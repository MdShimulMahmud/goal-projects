pipeline {
    agent none
    environment {
        GH_TOKEN = credentials('github-token')
        DOCKER_USERNAME = credentials('docker-username')
        DOCKER_PASSWORD = credentials('docker-password')
    }
    stages {
        stage('Backend') {
            agent any
            steps {
                script {
                    checkout scm
                    sh """
                        cd backend
                        npm install
                        echo "Run Number: ${BUILD_NUMBER}"
                        current_version=\$(cat ../VERSION)
                        echo "Current Version: \$current_version"
                    """
                }
            }
        }
        stage('Frontend') {
            agent any
            steps {
                script {
                    checkout scm
                    sh """
                        cd frontend
                        npm install
                        npm run build
                        echo "Run Number: ${BUILD_NUMBER}"
                        current_version=\$(cat ../VERSION)
                        echo "Current Version: \$current_version"
                    """
                }
            }
        }
        stage('Version Check') {
            agent any
            steps {
                script {
                    checkout scm
                    def previous_version = sh(
                        script: """
                            grep -oP 'shimulmahmud/(frontend|backend):v[0-9]+\\.[0-9]+\\.[0-9]+' k8s/deployment.yaml | grep -oP 'v[0-9]+\\.[0-9]+\\.[0-9]+' | head -n 1
                        """,
                        returnStdout: true
                    ).trim()
                    def current_version = sh(script: "cat VERSION", returnStdout: true).trim()
                    def version_diff = current_version != previous_version
                    if (!version_diff) {
                        error "No version difference detected. Skipping build."
                    }
                    currentBuild.description = "Current: ${current_version}, Previous: ${previous_version}"
                }
            }
        }
        stage('Docker') {
            agent any
            steps {
                script {
                    checkout scm
                    def current_version = sh(script: "cat VERSION", returnStdout: true).trim()
                    sh """
                        docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}
                        docker buildx create --use
                        docker buildx build ./frontend --push --tag ${DOCKER_USERNAME}/frontend:${current_version}
                        docker buildx build ./backend --push --tag ${DOCKER_USERNAME}/backend:${current_version}
                    """
                }
            }
        }
        stage('Update Kubernetes Tags') {
            agent any
            steps {
                script {
                    checkout scm
                    def current_version = sh(script: "cat VERSION", returnStdout: true).trim()
                    sh """
                        sed -i 's|shimulmahmud/frontend:v[0-9]*\\.[0-9]*\\.[0-9]*|shimulmahmud/frontend:${current_version}|' k8s/deployment.yaml
                        sed -i 's|shimulmahmud/backend:v[0-9]*\\.[0-9]*\\.[0-9]*|shimulmahmud/backend:${current_version}|' k8s/deployment.yaml
                        git config user.name "jenkins-bot"
                        git config user.email "jenkins@localhost"
                        git add k8s/deployment.yaml
                        git commit -m "Update Kubernetes deployment image tags"
                        git push origin master
                    """
                }
            }
        }
    }
}
