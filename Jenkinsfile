pipeline {
    agent any

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
