pipeline {
    agent any

    environment {
        // You can define your environment variables here later
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                echo "Starting Stage 1: Git Checkout..."
                checkout scm
                echo "Code checked out successfully."
            }
        }
    }
}
