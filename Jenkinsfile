pipeline {
    agent any

    environment {
        APP_NAME = "flask-app"
        IMAGE_NAME = "flask-devops"
        DOCKERHUB_IMAGE = "yourdockerhubusername/flask-devops"
        CONTAINER_NAME = "flask-container"
        STAGING_CONTAINER = "flask-staging"
        PORT = "5000"
        STAGING_PORT = "5001"
        BUILD_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Annsabsn/flask.git'
                echo "Code cloned successfully from GitHub"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:$BUILD_TAG .
                docker tag $IMAGE_NAME:$BUILD_TAG $IMAGE_NAME:latest
                '''
            }
        }

        stage('Run Basic Test') {
            steps {
                sh '''
                docker run --rm $IMAGE_NAME:$BUILD_TAG python -c "print('Flask container test successful')"
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker tag $IMAGE_NAME:$BUILD_TAG $DOCKERHUB_IMAGE:$BUILD_TAG
                    docker tag $IMAGE_NAME:$BUILD_TAG $DOCKERHUB_IMAGE:latest
                    docker push $DOCKERHUB_IMAGE:$BUILD_TAG
                    docker push $DOCKERHUB_IMAGE:latest
                    '''
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                sh '''
                docker stop $STAGING_CONTAINER || true
                docker rm $STAGING_CONTAINER || true

                docker run -d \
                    --name $STAGING_CONTAINER \
                    -p $STAGING_PORT:$PORT \
                    $IMAGE_NAME:$BUILD_TAG
                '''
            }
        }

        stage('Smoke Test Staging') {
            steps {
                sh '''
                sleep 5
                curl -f http://localhost:$STAGING_PORT/ || exit 1
                '''
            }
        }

        stage('Deploy to Production') {
            steps {
                input message: "Deploy build $BUILD_TAG to Production?"
                sh '''
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true

                docker run -d \
                    --name $CONTAINER_NAME \
                    -p $PORT:$PORT \
                    $IMAGE_NAME:$BUILD_TAG
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                docker ps
                curl -f http://localhost:$PORT/ || exit 1
                '''
            }
        }
    }

    post {
        success {
            echo "Application deployed successfully!"
            echo "Production URL: http://<your-server-ip>:5000"
            echo "Docker Image: $IMAGE_NAME:$BUILD_TAG"
        }

        failure {
            echo "Pipeline failed! Check logs immediately."
        }

        always {
            sh 'docker image prune -f || true'
        }
    }
}
