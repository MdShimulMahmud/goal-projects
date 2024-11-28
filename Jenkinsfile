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
        stage('Docker') {
            agent any
            steps {
                script {
                    checkout scm
                    def current_version = sh(script: "cat VERSION", returnStdout: true).trim()
                    try {
                        sh """
                            docker --version
                            docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}

                            docker build ./frontend --tag ${DOCKER_USERNAME}/frontend:${current_version}
                            docker push ${DOCKER_USERNAME}/frontend:${current_version}

                            docker build ./backend --tag ${DOCKER_USERNAME}/backend:${current_version}
                            docker push ${DOCKER_USERNAME}/backend:${current_version}
                        """
                    } finally {
                        echo "Image build failed"
                    }
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
                        git commit -m "Update Kubernetes deployment image tags to ${current_version}"
                        git push origin master
                    """
                }
            }
        }
    }
    post {
        always {
            echo "Pipeline completed."
        }
        failure {
            echo "Pipeline failed. Check the logs for errors."
        }
        success {
            echo "Pipeline succeeded."
        }
    }
}
