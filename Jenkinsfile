pipeline {
    agent any

    environment {
        ZAP_PORT = '8081'
        TARGET_URL = 'http://juice-shop:3001'
    }

    stages {
        stage('Start Juice Shop') {
            steps {
                script {
                    // Run Juice Shop container
                    docker.image('bkimminich/juice-shop').withRun('-p 3001:3000') { c ->
                        echo "Juice Shop is running at $TARGET_URL"
                    }
                }
            }
        }

        stage('Start OWASP ZAP') {
            steps {
                script {
                    // Run ZAP container
                    docker.image('zaproxy/zap-stable:latest').withRun("-u zap -p ${ZAP_PORT}:${ZAP_PORT} zaproxy/zap-stable:latest -daemon -host 0.0.0.0 -port ${ZAP_PORT}") { zap ->
                        env.ZAP_CONTAINER_ID = zap.id
                        echo "ZAP is running on port ${ZAP_PORT}"
                    }
                }
            }
        }

        stage('Run ZAP Security Scan') {
            steps {
                script {
                    // Ensure the container ID is correctly set
                    if (env.ZAP_CONTAINER_ID) {
                        // Run ZAP scan against Juice Shop
                        sh "docker exec ${env.ZAP_CONTAINER_ID} zap-cli quick-scan --self-contained ${TARGET_URL}"
                    } else {
                        error "ZAP container is not running"
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Clean up completed."
            // Optionally add cleanup steps here if needed, like stopping Docker containers
        }
    }
}
