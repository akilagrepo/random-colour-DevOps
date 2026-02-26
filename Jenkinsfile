pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "akilapradeep/react-jenkins"
        DOCKER_CREDENTIALS_ID = "akila-docker-hub-credentials"
        CONTAINER_NAME = "react-app"
        PORT = "80"
    }

    tools {
        nodejs 'Node18' // Make sure Node18 tool is configured in Jenkins
    }

    stages {

        stage('Clone Repository') {
            steps {
                sh '''
                rm -rf random-colour-DevOps || true
                git clone https://github.com/akilagrepo/random-colour-DevOps.git
                '''
            }
        }

        stage('Install Dependencies & Build React App') {
            steps {
                sh '''
                cd random-colour-DevOps
                npm install
                npm run build
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                cd random-colour-DevOps
                docker build -t $DOCKER_IMAGE .
                '''
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "$DOCKER_CREDENTIALS_ID",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy on EC2 (Run Container)') {
            steps {
                sh '''
                # Stop old container if exists
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true

                # Run new container
                docker run -d \
                  --name $CONTAINER_NAME \
                  -p $PORT:80 \
                  $DOCKER_IMAGE
                '''
            }
        }
    }

    post {
        success {
            echo "🚀 Deployment successful! Your app should be running on port $PORT"
        }
        failure {
            echo "❌ Deployment failed. Check the logs above."
        }
    }
}