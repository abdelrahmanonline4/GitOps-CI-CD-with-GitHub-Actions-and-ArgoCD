
pipeline {
    agent any

    options {
        timestamps()
    }

    environment {
        IMAGE_NAME = 'jenkinsfreestyle'
        CONTAINER_NAME = 'testcontainer'
        APP_PORT = '3000'
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                git branch: 'Master',
                    url: 'https://github.com/abdelrahmanonline4/GitOps-CI-CD-with-GitHub-Actions-and-ArgoCD.git'
            }
        }

        stage('Inspect Workspace') {
            steps {
                sh '''
                    pwd
                    ls -la
                    git rev-parse --short HEAD
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                        -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${APP_PORT}:${APP_PORT} \
                        ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Deployment Complete') {
            steps {
                sh '''
                    docker ps --filter "name=${CONTAINER_NAME}"
                    echo "Done deployment"
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins console output.'
        }
    }
}
