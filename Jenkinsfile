pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'
        DOCKER_IMAGE = "akhadekar07/react-advance-filtering"
        GIT_REPO = "https://github.com/abhishekkhadekar07/advance-filtering-jenkins-devops.git"
        GITHUB_TOKEN = credentials('github-token') 
        CONTAINER_NAME = "react-local-app"
        LOCAL_PORT = "3001"   // host port (Windows)
        CONTAINER_PORT = "80"  // app port inside container
    }

    stages {

        // stage('Clone Repository') {
        //     steps {
        //         echo "Cloning repository..."
                
        //         git branch: 'master', url: "${GIT_REPO}"
        //     }
        // }
         stage('Clone Repository') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: 'refs/heads/master']],
                    userRemoteConfigs: [[url: "${GIT_REPO}"]]
                ])
            }
        }
        stage('Install Dependencies') {
            steps {
                sh """
                    echo "Installing node modules..."
                    npm install
                """
            }
        }

        //  stage('Prettier Format Check (Non Blocking)') {
        //     steps {
        //         catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
        //             sh """
        //                 echo "Running Prettier Check..."
        //                 npm run format
        //             """
        //         }
        //     }
        // }
        stage('Prettier Format Check (Non Blocking)') {
            steps {
                script {
                    githubNotify context: "Prettier", description: "Checking code formatting...", status: "PENDING"

                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        sh "npm run format"
                    }

                    if (currentBuild.currentResult == "FAILURE") {
                        githubNotify context: "Prettier", description: "Formatting issues detected", status: "FAILURE"
                    } else {
                        githubNotify context: "Prettier", description: "Formatting OK", status: "SUCCESS"
                    }
                }
            }
        }

        // stage('ESLint Lint Check (Non Blocking)') {
        //     steps {
        //         catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
        //             sh """
        //                 echo "Running ESLint..."
        //                 npm run lint
        //             """
        //         }
        //     }
        // }

         stage('ESLint Lint Check (Non Blocking)') {
            steps {
                script {
                    githubNotify context: "ESLint", description: "Running ESLint...", status: "PENDING"

                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        sh "npm run lint"
                    }

                    if (currentBuild.currentResult == "FAILURE") {
                        githubNotify context: "ESLint", description: "Lint errors found", status: "FAILURE"
                    } else {
                        githubNotify context: "ESLint", description: "Linting passed", status: "SUCCESS"
                    }
                }
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
            echo "🎉 Build pushed and app running locally at http://localhost:3001"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
