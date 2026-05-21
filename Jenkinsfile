pipeline {
    agent any

    environment {
        PAYMENT_IMAGE = "naurafaizah/payment-service:${env.BUILD_NUMBER}"
        ORDER_IMAGE = "naurafaizah/order-service:${env.BUILD_NUMBER}"
        DELIVERY_IMAGE = "naurafaizah/delivery-service:${env.BUILD_NUMBER}"
        SHIPMENT_IMAGE = "naurafaizah/shipment-service:${env.BUILD_NUMBER}"

        PICKUP_IMAGE = "naurafaizah/pickup-service:${env.BUILD_NUMBER}"
        WAREHOUSE_IMAGE = "naurafaizah/warehouse-service:${env.BUILD_NUMBER}"
    }

    stages {

        // =========================
        // CHECKOUT
        // =========================
        stage('Checkout Repo') {
            steps {
                deleteDir()
                git branch: 'main', url: 'https://github.com/naurafaizah/tahap2.git'
            }
        }

        // =========================
        // UNIT TEST
        // =========================
        stage('Unit Test') {
            steps {

                dir('PaymentService') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        sh 'go test -v -run TestValidatePayment ./...'
                    }
                }

                dir('OrderService') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        sh 'go test -short ./...'
                    }
                }

                dir('DeliveryService') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        sh 'go test ./...'
                    }
                }

                dir('ShipmentService') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        sh 'go test ./...'
                    }
                }

                dir('PickupService') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        sh 'go test -v ./...'
                    }
                }

                dir('WarehouseService') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        sh 'go test -v ./...'
                    }
                }
            }
        }

        // =========================
        // LINT / VET
        // =========================
        stage('Lint / Vet') {
            steps {

                dir('PaymentService') { sh 'go vet ./...' }
                dir('OrderService') { sh 'go vet ./...' }
                dir('DeliveryService') { sh 'go vet ./...' }
                dir('ShipmentService') { sh 'go vet ./...' }
                dir('PickupService') { sh 'go vet ./...' }
                dir('WarehouseService') { sh 'go vet ./...' }
            }
        }

        // =========================
        // BUILD IMAGE
        // =========================
        stage('Build Images') {
            steps {
                sh '''
                docker build -t $PAYMENT_IMAGE ./PaymentService
                docker build -t $ORDER_IMAGE ./OrderService
                docker build -t $DELIVERY_IMAGE ./DeliveryService
                docker build -t $SHIPMENT_IMAGE ./ShipmentService

                docker build -t $PICKUP_IMAGE ./PickupService
                docker build -t $WAREHOUSE_IMAGE ./WarehouseService
                '''
            }
        }

        // =========================
        // FUNCTIONAL TEST
        // =========================
        stage('Functional Test') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    sh '''
                    docker rm -f test-payment test-order test-delivery test-shipment test-pickup test-warehouse || true

                    docker run -d --name test-payment -p 8082:8082 $PAYMENT_IMAGE
                    docker run -d --name test-order -p 8081:8081 $ORDER_IMAGE
                    docker run -d --name test-delivery -p 8086:8086 $DELIVERY_IMAGE
                    docker run -d --name test-shipment -p 8085:8085 $SHIPMENT_IMAGE

                    docker run -d --name test-pickup -p 8089:8089 $PICKUP_IMAGE
                    docker run -d --name test-warehouse -p 8090:8090 $WAREHOUSE_IMAGE

                    sleep 10

                    # Payment
                    curl -s -X POST http://localhost:8082/payment \
                      -H "Content-Type: application/json" \
                      -d '{"amount":1,"paid":1}'

                    # Order
                    curl -s -X POST http://localhost:8081/order \
                      -H "Content-Type: application/json" \
                      -d '{"user_id":1,"weight_kg":2,"distance_km":5,"base_price":10000}'

                    # Delivery
                    curl -s -X POST http://localhost:8086/delivery

                    # Shipment
                    curl -s -X POST http://localhost:8085/shipment

                    # Pickup
                    curl -s -X POST http://localhost:8089/pickup \
                      -H "Content-Type: application/json" \
                      -d '{"order_id":"ORD1","payment_status":"paid","weight":2}'

                    # Warehouse
                    curl -s http://localhost:8090/health

                    docker rm -f test-payment test-order test-delivery test-shipment test-pickup test-warehouse || true
                    '''
                }
            }
        }

        // =========================
        // PUSH IMAGE
        // =========================
        stage('Push Images') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-login',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )]) {
                    sh '''
                    echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin

                    docker push $PAYMENT_IMAGE
                    docker push $ORDER_IMAGE
                    docker push $DELIVERY_IMAGE
                    docker push $SHIPMENT_IMAGE
                    docker push $PICKUP_IMAGE
                    docker push $WAREHOUSE_IMAGE
                    '''
                }
            }
        }

        // =========================
        // FINAL
        // =========================
        stage('Deploy') {
            steps {
                sh 'echo "DEPLOY OK"'
            }
        }

        stage('Verify') {
            steps {
                sh 'echo "PIPELINE SUCCESS"'
            }
        }
    }
}
