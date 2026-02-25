pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "akilapradeep/react-random"
        DOCKER_CREDENTIALS_ID = "akila-docker-hub-credentials"
        CONTAINER_NAME = "react-app"
        PORT = "80"
    }

     tools {
        nodejs 'Node18'
    }


        stage('Test Docker Hub Login') {
            steps {
                withDockerRegistry([credentialsId: 'akila-docker-hub-credentials', url: 'https://index.docker.io/v1/']) {
                sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
                sh 'docker info'
                }
            }
        }

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
                withDockerRegistry([credentialsId: DOCKER_CREDENTIALS_ID, url: "https://index.docker.io/v1/"]) {
                    sh "docker push $DOCKER_IMAGE"
                }
            }
        }

        stage('Deploy on EC2 (Run Container)') {
            steps {
                sh '''
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true

                docker run -d \
                  --name $CONTAINER_NAME \
                  -p 80:80 \
                  $DOCKER_IMAGE
                '''
            }
        }
    }

    post {
        success {
            echo "🚀 Deployment successful!"
        }
        failure {
            echo "❌ Deployment failed. Check logs."
        }
    }
}