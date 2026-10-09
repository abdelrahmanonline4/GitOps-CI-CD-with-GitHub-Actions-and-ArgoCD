
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'jenkins-node-app'
        CONTAINER_NAME = 'jenkins-node-container'
        APP_PORT = '3000'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                      -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Docker Deploy') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      -p 127.0.0.1:${APP_PORT}:3000 \
                      ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Test Application') {
            steps {
                sh '''
                    for i in 1 2 3 4 5 6 7 8 9 10; do
                        if curl -fsS http://127.0.0.1:${APP_PORT}/ > /dev/null; then
                            echo "Application is responding"
                            exit 0
                        fi
                        sleep 2
                    done

                    echo "Application health check failed"
                    docker logs ${CONTAINER_NAME}
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed! Check Console Output.'
        }
    }
}
