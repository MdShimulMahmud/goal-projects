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
                            docker --version
                            docker info || echo "Docker daemon access may be restricted. Ensure user is in the docker group."
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
        stage('Synchronize Branches') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh """
                            git config user.name "jenkins-bot"
                            git config user.email "jenkins@localhost"
                            git checkout master
                            git pull --rebase https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/MdShimulMahmud/goal-projects master || true
                            sed -i 's|your_repo/frontend:v[0-9]*\\.[0-9]*\\.[0-9]*|your_repo/frontend:v1.0.3|' k8s/deployment.yaml
                            sed -i 's|your_repo/backend:v[0-9]*\\.[0-9]*\\.[0-9]*|your_repo/backend:v1.0.3|' k8s/deployment.yaml
                            git add k8s/deployment.yaml
                            git commit -m "Update Kubernetes deployment image tags to v1.0.3" || true
                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/MdShimulMahmud/goal-projects master
                        """
                    }
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
