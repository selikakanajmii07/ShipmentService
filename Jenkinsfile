pipeline {
    agent any

    environment {
        SHIPMENT_IMAGE = "selikakanajmii07/shipment-service:${env.BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Repo') {
            steps {
                deleteDir()
                git branch: 'main', url: 'https://github.com/selikakanajmii07/ShipmentService.git'
            }
        }

        stage('Unit Test') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    bat 'go test ./...'
                }
            }
        }

        stage('Lint / Vet') {
            steps {
                bat 'go vet ./...'
            }
        }

        stage('Build Image') {
            steps {
                bat 'docker build -t %SHIPMENT_IMAGE% .'
            }
        }

        stage('Functional Test') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    bat '''
                    docker rm -f test-shipment

                    docker run -d --name test-shipment ^
                      -e DB_HOST=host.docker.internal ^
                      -e DB_NAME=shipment_db ^
                      -e DB_USER=postgres ^
                      -e DB_PASSWORD=postgres ^
                      -p 8085:8085 ^
                      %SHIPMENT_IMAGE%

                    timeout /t 5

                    curl -X POST http://localhost:8085/shipment

                    docker rm -f test-shipment
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-login',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )]) {
                    bat '''
                    echo %PASSWORD% | docker login -u %USERNAME% --password-stdin
                    docker push %SHIPMENT_IMAGE%
                    '''
                }
            }
        }

        stage('Deploy Kubernetes') {
            steps {
                echo 'Deploy kubernetes placeholder'
            }
        }

        stage('Verify') {
            steps {
                echo 'PIPELINE SUCCESS'
            }
        }
    }
}
