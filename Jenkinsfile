pipeline {
    agent any 

    environment {
        DOCKERHUB_CREDENTIALS = credentials('testdock1')  
        IMAGE_NAME = "nemunis/mlapp"
    }

    stages {
        stage('Build Docker Image') {
            steps {  
                echo "Building Docker image..."
                sh "docker build -t $IMAGE_NAME:$BUILD_NUMBER ."
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'testdock1', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                    echo "Logging in to Docker Hub..."
                    sh "echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin"
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                echo "Pushing Docker image to Docker Hub..."
                sh "docker push $IMAGE_NAME:$BUILD_NUMBER"
                sh "docker tag $IMAGE_NAME:$BUILD_NUMBER $IMAGE_NAME:latest"
                sh "docker push $IMAGE_NAME:latest"
            }
        }

        stage('Deploy to Staging') {
            steps {
                echo "Deploying container..."
                // Pull the latest image
                sh "docker pull $IMAGE_NAME:$BUILD_NUMBER"

                // Stop and remove old container
                sh "docker stop mlapp-container || true"
                sh "docker rm mlapp-container || true"

                // Run new container (Streamlit runs on port 8501)
                sh "docker run -d --name mlapp-container -p 8501:8501 $IMAGE_NAME:$BUILD_NUMBER"
            }
        }
    }

    post {
        always {
            echo "Logging out from Docker Hub..."
            sh 'docker logout'
        }
    }
}
