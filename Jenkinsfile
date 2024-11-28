pipeline {
    agent none
    environment {
        GH_TOKEN = credentials('github-token')
        DOCKER_USERNAME = credentials('docker-username')
        DOCKER_PASSWORD = credentials('docker-password')
    }
    stages {
        stage('Checkout') {
            agent any
            when {
                beforeAgent true
            }
            steps {
                checkout scm
            }
        }
        stage('Backend') {
            agent any
            steps {
                script {
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
                    def current_version = sh(script: "cat VERSION", returnStdout: true).trim()
                    try {
                        sh """
                            docker --version
                            docker info || echo "Docker daemon access may be restricted. Ensure user is in the docker group."
                            docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}

                            docker build -t ${DOCKER_USERNAME}/frontend:${current_version} ./frontend
                            docker push ${DOCKER_USERNAME}/frontend:${current_version}

                            docker build -t ${DOCKER_USERNAME}/backend:${current_version} ./backend
                            docker push ${DOCKER_USERNAME}/backend:${current_version}
                        """
                    } catch (Exception e) {
                        error "Docker build or push failed: ${e}"
                    }
                }
            }
        }
        stage('Update Kubernetes Tags') {
            agent any
            steps {
                script {
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
