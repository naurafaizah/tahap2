pipeline {
    agent any

    environment {
        IMAGE = "nadzallad/pickup-service:${env.BUILD_NUMBER}"
    }

    stages {

        // 1. CHECKOUT
        stage('Checkout Repo') {
            steps {
                deleteDir()
                git branch: 'main', url: 'https://github.com/nadzallad/Cloud2.git'
            }
        }

        // 2. UNIT TEST
        stage('Unit Test') {
            steps {
                dir('PickupService') {
                    bat 'go test ./...'
                }
            }
        }

        // 3. LINT / VET
        stage('Lint / Vet') {
            steps {
                dir('PickupService') {
                    bat 'go vet ./...'
                }
            }
        }

        // 4. BUILD IMAGE
        stage('Build Image') {
            steps {
                bat 'docker build -t %IMAGE% ./PickupService'
            }
        }

        // 5. FUNCTIONAL TEST
        stage('Functional Test') {
            steps {
                bat '''
                docker run -d -p 8083:8083 --name test-pickup %IMAGE%
                timeout /t 5

                curl -X POST http://localhost:8083/pickup ^
                -H "Content-Type: application/json" ^
                -d "{\\"order_id\\":1,\\"courier_name\\":\\"Budi\\",\\"status\\":\\"waiting pickup\\"}"

                docker stop test-pickup
                docker rm test-pickup
                '''
            }
        }

        // 6. PUSH IMAGE
        stage('Push Image') {
            steps {
                bat 'docker push %IMAGE%'
            }
        }

        // 7. DEPLOY KUBERNETES
        stage('Deploy') {
            steps {
                bat 'kubectl apply -f k8s/'
            }
        }

        // 8. VERIFY
        stage('Verify') {
            steps {
                bat 'kubectl get pods'
                bat 'kubectl get svc'
            }
        }
    }
}