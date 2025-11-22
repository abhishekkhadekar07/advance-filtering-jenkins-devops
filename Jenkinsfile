pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'
        DOCKER_IMAGE = "akhadekar07/react-advance-filtering"
        GIT_REPO = "https://github.com/abhishekkhadekar07/advance-filtering-jenkins-devops.git"
        CONTAINER_NAME = "react-local-app"
        LOCAL_PORT = "3001"   // host port (Windows)
        CONTAINER_PORT = "80"  // app port inside container
    }

    stages {

        stage('Clone Repository') {
            steps {
                echo "Cloning repository..."
                git branch: 'master', url: "${GIT_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    echo "Building Docker image..."
                    docker build -t ${DOCKER_IMAGE}:latest .
                """
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh """
                    echo "Pushing latest image..."
                    docker push ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Stop Old Container') {
            steps {
                sh """
                    echo "Stopping old container if running..."
                    docker stop ${CONTAINER_NAME} || true

                    echo "Removing old container..."
                    docker rm ${CONTAINER_NAME} || true
                """
            }
        }

        stage('Run New Container Locally') {
            steps {
                sh """
                    echo "Running new container on port 3000..."
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${LOCAL_PORT}:${CONTAINER_PORT} \
                        ${DOCKER_IMAGE}:latest

                    echo "Application now live at: http://localhost:${LOCAL_PORT}"
                """
            }
        }
    }

    post {
        success {
            echo "🎉 Build pushed and app running locally at http://localhost:3000"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
