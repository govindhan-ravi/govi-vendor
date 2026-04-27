pipeline {
    agent any

    tools {
        // Requires 'NodeJS' plugin and a tool named 'node' configured in Jenkins Global Tool Configuration
        nodejs 'node'
    }

    stages {
        stage('Stage 1: Git Checkout') {
            steps {
                echo "Starting Stage 1: Git Checkout..."
                checkout scm
                echo "Code checked out successfully."
            }
        }

        stage('Stage 2: ESLint (Code Quality Check)') {
            steps {
                echo "Starting Stage 2: ESLint..."
                dir('frontend') {
                    sh 'npm ci'
                    sh 'npm run lint'
                }
                echo "ESLint checks passed successfully."
            }
        }

        stage('Stage 3: Frontend Tests') {
            steps {
                echo "Starting Stage 3: Frontend Tests..."
                dir('frontend') {
                    sh 'npm test'
                }
            }
        }

        stage('Stage 4: Backend Tests') {
            steps {
                echo "Starting Stage 4: Backend Tests..."
                dir('backend') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }

        stage('Stage 5: SonarQube Scan') {
            environment {
                // Requires 'SonarQube Scanner' plugin and a tool named 'sonar-scanner' configured in Jenkins
                SCANNER_HOME = tool 'sonar-scanner'
            }
            steps {
                echo "Starting Stage 5: SonarQube Scan..."
                // For a real SonarQube server, you would wrap this in withSonarQubeEnv('your-server-name')
                // This basic command assumes SonarQube is running locally or configured globally.
                sh "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=vendor-management-system -Dsonar.projectName=Vendor-Management -Dsonar.sources=."
            }
        }
    }
}
