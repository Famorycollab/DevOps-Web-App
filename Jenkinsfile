
pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t devops-web-app:latest .'
            }
        }

        stage('Deploy Application') {
            steps {
                bat '''
                    docker rm -f devops-web-container 2>NUL
                    docker run -d --name devops-web-container -p 8080:80 devops-web-app:latest
                '''
            }
        }
    }
}
