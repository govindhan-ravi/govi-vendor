pipeline {
    agent any

    tools {
        // Requires 'NodeJS' plugin and a tool named 'node' configured in Jenkins Global Tool Configuration
        nodejs 'node'
    }

    stages {
        stage('Git Checkout') {
            steps {
                echo "Starting Stage 1: Git Checkout..."
                checkout scm
                echo "Code checked out successfully."
            }
        }

        stage('ESLint (Code Quality Check)') {
            steps {
                echo "Starting Stage 2: ESLint..."
                dir('frontend') {
                    // Install dependencies and run linting
                    sh 'npm ci'
                    sh 'npm run lint'
                }
                echo "ESLint checks passed successfully."
            }
        }

        stage('Frontend Tests') {
            steps {
                echo "Starting Stage 3: Frontend Tests..."
                dir('frontend') {
                    sh 'npm test'
                }
            }
        }

        stage('Backend Tests') {
            steps {
                echo "Starting Stage 4: Backend Tests..."
                dir('backend') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }
    }
}
