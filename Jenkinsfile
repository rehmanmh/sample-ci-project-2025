pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'M3'
    }

    environment {
        JAVA_HOME = tool 'JDK17'
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
        DOCKER_IMAGE = "sample-ci-project"
        DOCKER_HOST_IP = "13.48.196.47"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/rehmanmh/sample-ci-project-2025.git'
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Run Tests') {
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
                sshagent(credentials: ['docker-host']) {
                    sh """
                        scp -o StrictHostKeyChecking=no target/*.jar ec2-user@${DOCKER_HOST_IP}:/home/ec2-user/ci-build/app.jar
                        scp -o StrictHostKeyChecking=no Dockerfile ec2-user@${DOCKER_HOST_IP}:/home/ec2-user/ci-build/Dockerfile
                    """
                }
            }
        }

        stage('Build Docker Image on Remote Host') {
            steps {
                sshagent(credentials: ['docker-host']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${DOCKER_HOST_IP} '
                            cd /home/ec2-user/ci-build &&
                            docker build -t ${DOCKER_IMAGE}:latest . &&
                            echo "Docker image built successfully 🚀"
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✔ Full CI + Docker automation completed! 🚀"
        }
        failure {
            echo "❌ Pipeline failed — check logs."
        }
    }
}
