pipeline {
    agent any
    
    environment {
        ZAP_PORT = '8080'
        TARGET_URL = 'http://juice-shop:3000'
    }
    
    stages {
        stage('Start Juice Shop') {
            steps {
                script {
                    // Run Juice Shop container
                    docker.image('bkimminich/juice-shop').withRun('-p 3000:3000') { c ->
                        echo "Juice Shop is running at $TARGET_URL"
                    }
                }
            }
        }
        
        stage('Start OWASP ZAP') {
            steps {
                script {
                    // Run ZAP container
                    docker.image('owasp/zap2docker-stable').withRun('-u zap -p 8080:8080 zap.sh -daemon -host 0.0.0.0 -port 8080') { zap ->
                        env.ZAP_CONTAINER_ID = zap.id
                        echo "ZAP is running on port ${ZAP_PORT}"
                    }
                }
            }
        }
        
        stage('Run ZAP Security Scan') {
            steps {
                script {
                    // Run ZAP scan against Juice Shop
                    sh "docker exec ${env.ZAP_CONTAINER_ID} zap-cli quick-scan --self-contained ${TARGET_URL}"
                }
            }
        }
    }
    
    post {
        always {
            echo "Clean up completed."
        }
    }
}
