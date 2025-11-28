pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'M3'
    }

    environment {
        JAVA_HOME = tool 'JDK17'
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
        DOCKER_HOST_IP = "16.16.122.248"           // your Docker Host
        NEXUS_IP = "16.16.70.183:5000"          // Nexus Docker Registry
        DOCKER_IMAGE = "sample-ci-project"       // Image name
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/rehmanmh/sample-ci-project-2025.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('prod-sonar') {
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=sample-ci-project-2025 \
                        -Dsonar.projectName=sample-ci-project-2025
                    """
                }
            }
        }

        stage('Copy Artifact to Docker Host') {
            steps {
                sshagent (credentials: ['docker-host']) {
                    sh """
                        scp -o StrictHostKeyChecking=no target/sample-ci-project-1.0.0.jar ec2-user@${DOCKER_HOST_IP}:/home/ec2-user/ci-build/app.jar
                        scp -o StrictHostKeyChecking=no Dockerfile ec2-user@${DOCKER_HOST_IP}:/home/ec2-user/ci-build/Dockerfile
                    """
                }
            }
        }

        stage('Build & Push Docker Image from Docker Host') {
            steps {
                sshagent (credentials: ['docker-host']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${DOCKER_HOST_IP} '
                            cd /home/ec2-user/ci-build &&
                            docker build -t ${DOCKER_IMAGE}:latest . &&
                            docker tag ${DOCKER_IMAGE}:latest ${NEXUS_IP}/${DOCKER_IMAGE}:latest &&
                            docker push ${NEXUS_IP}/${DOCKER_IMAGE}:latest &&
                            echo "🚀 Docker image pushed to Nexus: ${NEXUS_IP}/${DOCKER_IMAGE}:latest"
                        '
                    """
                }
            }
        }

    }

    post {
        success {
            echo "✔ Full CI → Docker Build → Nexus Push Completed Successfully!"
        }
        failure {
            echo "❌ Pipeline failed — check logs."
        }
    }
}
