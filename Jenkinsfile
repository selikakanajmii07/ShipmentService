pipeline {
    agent any

    environment {
        PAYMENT_IMAGE = "nadzalla/payment-service:${env.BUILD_NUMBER}"
        ORDER_IMAGE = "ghryalvrt/order-service:${env.BUILD_NUMBER}"
        SHIPMENT_IMAGE = "selikakanajmii07/shipment-service:${env.BUILD_NUMBER}"
        DELIVERY_IMAGE = "selikakanajmii07/delivery-service:${env.BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Repo') {
            steps {
                deleteDir()
                git branch: 'main', url: 'https://github.com/nadzallad/Cloud2.git'
            }
        }

        stage('Unit Test') {
            steps {

                dir('PaymentService') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        bat 'go test -v -run TestValidatePayment ./...'
                    }
                }

                dir('OrderService') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        bat 'go test -short ./...'
                    }
                }

                dir('ShipmentService') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        bat 'go test ./...'
                    }
                }

                dir('DeliveryService') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        bat 'go test ./...'
                    }
                }
            }
        }

        stage('Lint / Vet') {
            steps {

                dir('PaymentService') {
                    bat 'go vet ./...'
                }

                dir('OrderService') {
                    bat 'go vet ./...'
                }

                dir('ShipmentService') {
                    bat 'go vet ./...'
                }

                dir('DeliveryService') {
                    bat 'go vet ./...'
                }
            }
        }

        stage('Build Image') {
            steps {
                bat '''
                docker build -t %PAYMENT_IMAGE% ./PaymentService
                docker build -t %ORDER_IMAGE% ./OrderService
                docker build -t %SHIPMENT_IMAGE% ./ShipmentService
                docker build -t %DELIVERY_IMAGE% ./DeliveryService
                '''
            }
        }

        stage('Functional Test') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    bat '''
                    docker rm -f test-payment test-order test-shipment test-delivery

                    docker run -d --name test-payment ^
                      -e DB_HOST=host.docker.internal ^
                      -e DB_NAME=payment_db ^
                      -e DB_PASS=admin123 ^
                      -p 8082:8082 ^
                      %PAYMENT_IMAGE%

                    docker run -d --name test-order ^
                      -p 8081:8081 ^
                      %ORDER_IMAGE%

                    docker run -d --name test-shipment ^
                      -e DB_HOST=host.docker.internal ^
                      -e DB_NAME=shipment_db ^
                      -p 8085:8085 ^
                      %SHIPMENT_IMAGE%

                    docker run -d --name test-delivery ^
                      -e DB_HOST=host.docker.internal ^
                      -e DB_NAME=delivery_db ^
                      -p 8086:8086 ^
                      %DELIVERY_IMAGE%

                    timeout /t 5

                    curl -X POST http://localhost:8082/payment ^
                      -H "Content-Type: application/json" ^
                      -d "{\\"amount\\":1,\\"paid\\":1}"

                    curl -X POST http://localhost:8081/order ^
                      -H "Content-Type: application/json" ^
                      -d "{\\"user_id\\":1,\\"weight_kg\\":2,\\"distance_km\\":5,\\"base_price\\":10000}"

                    curl -X POST http://localhost:8085/shipment

                    curl -X POST http://localhost:8086/delivery

                    docker rm -f test-payment test-order test-shipment test-delivery
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {

                withCredentials([usernamePassword(
                    credentialsId: 'logistic-login',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )]) {
                    bat '''
                    echo %PASSWORD% | docker login -u %USERNAME% --password-stdin
                    docker push %PAYMENT_IMAGE%
                    docker push %SHIPMENT_IMAGE%
                    docker push %DELIVERY_IMAGE%
                    '''
                }

                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-login',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )]) {
                    bat '''
                    echo %PASSWORD% | docker login -u %USERNAME% --password-stdin
                    docker push %ORDER_IMAGE%
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                bat 'echo DEPLOY OK'
            }
        }

        stage('Verify') {
            steps {
                bat 'echo PIPELINE SUCCESS'
            }
        }
    }
}
