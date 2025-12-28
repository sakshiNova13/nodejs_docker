pipeline {
    agent any

    environment {
        IMAGE_NAME = "sakshinova/nodejs2"
        IMAGE_TAG = "latest"
        CONTAINER_NAME = "nodejs-app"
        REMOTE_USER = "ubuntu"
        REMOTE_HOST = "172.31.44.49"
        SSH_KEY_ID = "ssh-key"    // Jenkins Credentials ID for SSH key
    }

    stages {
        stage('Checkout') {
            steps {
                echo "🔁 Cloning source code..."
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "📦 Building Docker image..."
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    echo "📤 Logging in to Docker Hub..."
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    echo "🚀 Pushing image to Docker Hub..."
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo "🔒 Connecting to remote EC2 to deploy..."
                sshagent([SSH_KEY_ID]) {
                   // Remove old container
                    sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} 'docker rm -f ${CONTAINER_NAME} || true'"

                    // Pull latest image
                    sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} 'docker pull ${IMAGE_NAME}:${IMAGE_TAG}'"

                    // Run container
                    sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} 'docker run -d --name ${CONTAINER_NAME} -p 3000:3000 ${IMAGE_NAME}:${IMAGE_TAG}'"

                    // Optional success echo
                    sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} 'echo ✅ Deployment successful!'"
                }
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD pipeline completed successfully."
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
