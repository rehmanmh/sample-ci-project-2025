pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'M3'
    }

    environment {
        JAVA_HOME = tool 'JDK17'
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/rehmanmh/sample-ci-project-2025.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn -v'
                sh 'java -version'
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

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        success {
            echo "Build + Sonar scan completed successfully! 🚀"
        }
        failure {
            echo "Pipeline failed ❌ Check logs."
        }
    }
}


