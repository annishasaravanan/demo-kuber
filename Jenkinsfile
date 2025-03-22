pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "paddy1123/docker-app:latest"  // Change this to your registry
        DOCKER_IMAGE_2 = "paddy1123/another-docker-app:latest"  // Second Docker image
        CONTAINER_NAME = "docker-running-app"
        REGISTRY_CREDENTIALS = "demo-docker"  // Jenkins credentials ID
    }

    stages {
        stage('Checkout Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'git-demo', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    git url: "https://$GIT_USER:$GIT_TOKEN@github.com/annishasaravanan/demo-kuber.git", branch: 'main'
                }
            }
        }

        stage('Build Docker Image 1') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Build Docker Image 2') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE_2 .'
            }
        }

        stage('Login to Docker Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'demo-docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push to Container Registry - Image 1') {
            steps {
                sh 'docker push $DOCKER_IMAGE'
            }
        }

        stage('Push to Container Registry - Image 2') {
            steps {
                sh 'docker push $DOCKER_IMAGE_2'
            }
        }

        stage('Stop & Remove Existing Container') {
            steps {
                script {
                    sh '''
                    if [ "$(docker ps -aq -f name=$CONTAINER_NAME)" ]; then
                        docker stop $CONTAINER_NAME || true
                        docker rm $CONTAINER_NAME || true
                    fi
                    '''
                }
            }
        }

        stage('Run Docker Container for Image 1') {
            steps {
                sh 'docker run -d -p 5001:5000 --name $CONTAINER_NAME $DOCKER_IMAGE'
            }
        }

        stage('Run Docker Container for Image 2') {
            steps {
                sh 'docker run -d -p 5002:5000 --name ${CONTAINER_NAME}_2 $DOCKER_IMAGE_2'
            }
        }
    }

    post {
        success {
            echo "Build, push, and container execution successful!"
        }
        failure {
            echo "Build or container execution failed."
        }
    }
}

