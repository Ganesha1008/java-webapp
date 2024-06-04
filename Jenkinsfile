pipeline {
    agent any

    environment {
        // Retrieve DockerHub credentials securely using Secret Manager (replace with your actual secret ID)
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-token')
        REMOTE_SERVER = '65.2.170.193'
        REMOTE_USER = 'ubuntu'
    }

    // Fetch code from GitHub
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Ganesha1008/java-webapp.git'
            }
        }

        // Build Java application (assuming Maven project structure)
        stage('Maven Build') {
            steps {
                sh 'mvn clean package' // Use 'package' goal for building JAR
            }

            // Archive built JAR for reference (optional)
            post {
                success {
                    archiveArtifacts artifacts: '**/target/*.jar'
                }
            }
        }

        // Test Java application (assuming Maven project structure)
        stage('Maven Test') {
            steps {
                sh 'mvn test'
            }
        }

        // Build Docker image in Jenkins
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t java-webapp:latest .'
                sh 'docker tag java-webapp sunilmargale/java-webapp:latest' // Consider using a private DockerHub repository for security
            }
        }

        // Login to DockerHub before pushing (secure approach using credentials)
        stage('Login to DockerHub') {
            steps {
                sh 'docker login -u $DOCKERHUB_CREDENTIALS_USR -p $DOCKERHUB_CREDENTIALS_PSW'
            }
        }

        // Push image to DockerHub registry
        stage('Push Image to DockerHub') {
            steps {
                sh 'docker push sunilmargale/java-webapp:latest' // Consider using a private DockerHub repository for security
            }

            // Logout after pushing (optional but recommended)
            post {
                always {
                    sh 'docker logout'
                }
            }
        }

        // Deploy Docker image to AWS instance (improved security and error handling)
        stage('Deploy Docker image to AWS instance') {
            steps {
                script {
                    sshagent(credentials: ['awscred']) {
                        def remoteHost = "${REMOTE_USER}@${REMOTE_SERVER}"

                        // Check connection and handle potential errors
                        sh "ssh -o StrictHostKeyChecking=no ${remoteHost} 'exit 0' || echo 'Failed to connect to AWS instance. Check SSH configuration.'"

                        // Stop and remove any existing container named 'javaApp' (handle potential errors)
                        sh """
                            ssh -o StrictHostKeyChecking=no ${remoteHost} 'docker stop javaApp || true && docker rm javaApp || true' || echo 'Failed to stop/remove existing container "javaApp".'
                        """

                        // Pull the latest image from DockerHub
                        sh "ssh -o StrictHostKeyChecking=no ${remoteHost} 'docker pull sunilmargale/java-webapp:latest'"

                        // Run the container with proper detachment (-d) and port mapping (8081:8081)
                        sh "ssh -o StrictHostKeyChecking=no ${remoteHost} 'docker run --name javaApp -d -p 8081:8081 sunilmargale/java-webapp:latest'"
                    }
                }
            }
        }
    }
}
