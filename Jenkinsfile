pipeline {
    agent none
    environment {
        GH_TOKEN = credentials('github-token')  // GitHub token
        DOCKER_USERNAME = credentials('docker-username')  // Docker Hub username
        DOCKER_PASSWORD = credentials('docker-password')  // Docker Hub access token
    }
    stages {
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
                            echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin

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
        stage('Trigger Update Manifest Pipeline') {
            agent any
            steps {
                script {
                    def current_version = sh(script: "cat VERSION", returnStdout: true).trim()
                    
                    build job: 'update-k8s-manifests', parameters: [
                        string(name: 'IMAGE_TAG', value: "${current_version}")
                    ]
                }
            }
        }

    }
}
