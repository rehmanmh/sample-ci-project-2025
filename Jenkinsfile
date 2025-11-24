pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'M3'
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
    }

    post {
        success {
            echo "Build completed successfully! 🚀"
        }
        failure {
            echo "Build failed ❌ Check console output."
        }
    }
}
