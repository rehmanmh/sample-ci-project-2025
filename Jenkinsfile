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

        stage('Build Docker Image') {
            steps {
                script {
                    def tag = "${env.BUILD_NUMBER}"
                    sh "docker build -t ${DOCKER_IMAGE}:latest ."
                    sh "docker tag ${DOCKER_IMAGE}:latest ${DOCKER_IMAGE}:${tag}"
                }
            }
        }
    }

    post {
        success {
            echo "Build + Sonar + Docker completed successfully! 🚀"
        }
        failure {
            echo "Pipeline failed ❌ Check logs."
        }
    }
}
