
pipeline {
    agent any

    options {
        timestamps()
        skipDefaultCheckout(true)
        disableConcurrentBuilds()
    }

    parameters {
        choice(
            name: 'ACTION',
            choices: ['DEPLOY', 'ROLLBACK'],
            description: 'Select Deploy or Rollback'
        )

        string(
            name: 'ROLLBACK_VERSION',
            defaultValue: '',
            description: 'Previous Docker Image tag (e.g. 10)'
        )
    }

    environment {
        IMAGE_NAME = 'jenkinsfreestyle'
        CONTAINER_NAME = 'testcontainer'
        APP_PORT = '3000'
    }

    stages {

        stage('Checkout Source Code') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                git branch: 'Master',
                    url: 'https://github.com/abdelrahmanonline4/GitOps-CI-CD-with-GitHub-Actions-and-ArgoCD.git'
            }
        }

        stage('Inspect Workspace') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                sh '''
                    pwd
                    ls -la
                    git rev-parse --short HEAD
                '''
            }
        }

        stage('Build Docker Image') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                sh '''
                    docker build \
                        -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Production Approval Gate') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    input(
                        message: 'Approve Production Deployment?',
                        ok: 'Approve Deployment'
                    )
                }

                echo 'Deployment Approved!'
            }
        }

        stage('Deploy Container') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
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

        stage('Rollback Previous Version') {
            when {
                expression { params.ACTION == 'ROLLBACK' }
            }
            steps {
                script {
                    def version = params.ROLLBACK_VERSION.trim()

                    if (!(version ==~ /^[0-9]+$/)) {
                        error('Enter a valid numeric ROLLBACK_VERSION.')
                    }

                    withEnv(["TARGET_VERSION=${version}"]) {
                        sh '''
                            set -eu

                            echo "Checking previous Docker Image..."

                            docker image inspect \
                                "${IMAGE_NAME}:${TARGET_VERSION}" \
                                > /dev/null

                            echo "Rolling back to version ${TARGET_VERSION}"

                            docker rm -f "${CONTAINER_NAME}" || true

                            docker run -d \
                                --name "${CONTAINER_NAME}" \
                                -p "${APP_PORT}:${APP_PORT}" \
                                "${IMAGE_NAME}:${TARGET_VERSION}"

                            echo "Rollback container started."
                        '''
                    }
                }
            }
        }

        stage('Deployment Complete') {
            steps {
                sh '''
                    docker ps --filter "name=${CONTAINER_NAME}"
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check Jenkins console output.'
        }

        aborted {
            echo 'Pipeline aborted.'
        }
    }
}
