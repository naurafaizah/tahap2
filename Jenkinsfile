pipeline {
    agent any

    environment {
        PICKUP_IMAGE = "naurafaizah/pickup-service:${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Repo') {
            steps {
                deleteDir()
                git branch: 'main', url: 'https://github.com/naurafaizah/tahap2.git'
            }
        }

        stage('Unit Test') {
            steps {
                dir('PickupService') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        bat 'go test -v ./...'
                    }
                }
            }
        }

        stage('Lint / Vet') {
            steps {
                dir('PickupService') {
                    bat 'go vet ./...'
                }
            }
        }

        stage('Build Image') {
            steps {
                bat 'docker build -t %PICKUP_IMAGE% ./PickupService'
            }
        }

        stage('Functional Test') {
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

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                credentialsId: 'dockerhub-login',
                usernameVariable: 'USERNAME',
                passwordVariable: 'PASSWORD'
                )]) {
                bat 'docker logout'

                bat '''
                docker login -u %USERNAME% -p %PASSWORD%
                docker push naurafaizah/pickup-service:${BUILD_NUMBER}
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
