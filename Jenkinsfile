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
                SCANNER_HOME = tool 'sonar-scanner'
            }
            steps {
                echo "Starting Stage 5: SonarQube Scan..."
                withSonarQubeEnv('sonar-server') {
                    sh "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=vendor-management-system -Dsonar.projectName=Vendor-Management -Dsonar.sources=."
                }
            }
        }

        stage('Stage 6: Quality Gate') {
            steps {
                echo "Starting Stage 6: Quality Gate..."
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Stage 7: Trivy Filesystem Scan') {
            steps {
                echo "Starting Stage 7: Trivy Filesystem Scan..."
                // Scans the current directory for vulnerabilities and outputs to a text file
                sh 'trivy fs . > trivy_fs_report.txt'
            }
        }

        stage('Stage 8: Docker Build') {
            steps {
                echo "Starting Stage 8: Docker Build..."
                // Build images and tag them with the Jenkins Build Number
                sh "docker build -t vendor-backend:${env.BUILD_NUMBER} ./backend"
                sh "docker build -t vendor-frontend:${env.BUILD_NUMBER} ./frontend"
            }
        }

        stage('Stage 9: Trivy Image Scan') {
            steps {
                echo "Starting Stage 9: Trivy Image Scan..."
                // Scans the newly built Docker images for vulnerabilities
                sh "trivy image vendor-backend:${env.BUILD_NUMBER} > trivy_backend_image_report.txt"
                sh "trivy image vendor-frontend:${env.BUILD_NUMBER} > trivy_frontend_image_report.txt"
            }
        }

        stage('Stage 10: Docker Push') {
            steps {
                echo "Starting Stage 10: Docker Push..."
                // Uses the 'docker-hub' credentials we created in Jenkins
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                    
                    // Tag images with your Docker Hub username
                    sh "docker tag vendor-backend:${env.BUILD_NUMBER} ${DOCKER_USERNAME}/vendor-backend:${env.BUILD_NUMBER}"
                    sh "docker tag vendor-frontend:${env.BUILD_NUMBER} ${DOCKER_USERNAME}/vendor-frontend:${env.BUILD_NUMBER}"
                    sh "docker tag vendor-backend:${env.BUILD_NUMBER} ${DOCKER_USERNAME}/vendor-backend:latest"
                    sh "docker tag vendor-frontend:${env.BUILD_NUMBER} ${DOCKER_USERNAME}/vendor-frontend:latest"
                    
                    // Push images to Docker Hub
                    sh "docker push ${DOCKER_USERNAME}/vendor-backend:${env.BUILD_NUMBER}"
                    sh "docker push ${DOCKER_USERNAME}/vendor-frontend:${env.BUILD_NUMBER}"
                    sh "docker push ${DOCKER_USERNAME}/vendor-backend:latest"
                    sh "docker push ${DOCKER_USERNAME}/vendor-frontend:latest"
                }
            }
        }

        stage('Stage 11: Helm Lint') {
            steps {
                echo "Starting Stage 11: Helm Lint..."
                // Verify the newly created Helm charts are valid
                sh 'helm lint ./helm/vendor-management'
            }
        }

        stage('Stage 12: EKS Authentication') {
            steps {
                echo "Starting Stage 12: EKS Authentication..."
                // Uses the EC2 IAM Instance Role (Jenkins-EKS-Admin) - no credentials needed!
                sh 'aws eks update-kubeconfig --region us-east-1 --name vendor-cluster'
            }
        }

        stage('Stage 13: Helm Deploy') {
            steps {
                echo "Starting Stage 13: Helm Deploy..."
                // Uses the EC2 IAM Instance Role directly - no credentials needed!
                sh '''
                aws eks update-kubeconfig --region us-east-1 --name vendor-cluster
                helm upgrade --install vendor-management ./helm/vendor-management \
                  --set image.repository=govindhan1234 \
                  --set image.tag=''' + env.BUILD_NUMBER + '''
                '''
            }
        }
    }
}

