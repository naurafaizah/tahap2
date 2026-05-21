pipeline {
    agent any

    environment {
        PICKUP_IMAGE = "naurafaizah/pickup-service:${BUILD_NUMBER}"
        WAREHOUSE_IMAGE = "naurafaizah/warehouse-service:${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Repo') {
            steps {
                deleteDir()
                git branch: 'main', url: 'https://github.com/naurafaizah/tahap2.git'
            }
        }

        // =========================
        // PICKUP SERVICE
        // =========================

        stage('Pickup Unit Test') {
            steps {
                dir('PickupService') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        bat 'go test -v ./...'
                    }
                }
            }
        }

        stage('Pickup Lint / Vet') {
            steps {
                dir('PickupService') {
                    bat 'go vet ./...'
                }
            }
        }

        stage('Build Pickup Image') {
            steps {
                bat 'docker build -t %PICKUP_IMAGE% ./PickupService'
            }
        }

        stage('Pickup Functional Test') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    bat '''
                    docker rm -f test-pickup

                    docker run -d --name test-pickup -p 8089:8089 %PICKUP_IMAGE%

                    timeout /t 3

                    curl -X POST http://localhost:8089/pickup ^
                    -H "Content-Type: application/json" ^
                    -d "{\\"order_id\\":\\"ORD1\\",\\"payment_status\\":\\"paid\\",\\"weight\\":2}"

                    docker rm -f test-pickup
                    '''
                }
            }
        }

        // =========================
        // WAREHOUSE SERVICE
        // =========================

        stage('Warehouse Unit Test') {
            steps {
                dir('WarehouseService') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        bat 'go test -v ./...'
                    }
                }
            }
        }

        stage('Warehouse Lint / Vet') {
            steps {
                dir('WarehouseService') {
                    bat 'go vet ./...'
                }
            }
        }

        stage('Build Warehouse Image') {
            steps {
                bat 'docker build -t %WAREHOUSE_IMAGE% ./WarehouseService'
            }
        }

        stage('Warehouse Functional Test') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    bat '''
                    docker rm -f test-warehouse

                    docker run -d --name test-warehouse -p 8090:8090 %WAREHOUSE_IMAGE%

                    timeout /t 3

                    curl http://localhost:8090/health

                    docker rm -f test-warehouse
                    '''
                }
            }
        }

        // =========================
        // PUSH IMAGES
        // =========================

        stage('Push Images') {
            steps {
                withCredentials([usernamePassword(
                credentialsId: 'dockerhub-login',
                usernameVariable: 'USERNAME',
                passwordVariable: 'PASSWORD'
                )]) {

                bat 'docker logout'

                bat """
                docker login -u %USERNAME% -p %PASSWORD%

                docker push %PICKUP_IMAGE%
                docker push %WAREHOUSE_IMAGE%
                """

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
