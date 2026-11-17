
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        DOCKER = 'C:\\Users\\Famory Toure\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        IMAGE = 'famorytoure/devops-web-app'
        CONTAINER = 'devops-web-app-smoke'
    }

    stages {

        stage('1 - Checkout') {
            steps {
                checkout scm
                echo 'Code source recupere avec succes.'
            }
        }

        stage('2 - Docker Build') {
            steps {
                bat '''
                    "%DOCKER%" build -t %IMAGE%:build-%BUILD_NUMBER% .
                    "%DOCKER%" tag %IMAGE%:build-%BUILD_NUMBER% %IMAGE%:latest
                '''
            }
        }

        stage('3 - Smoke Test') {
            steps {
                bat '''
                    "%DOCKER%" rm -f %CONTAINER% 2>NUL
                    "%DOCKER%" run -d --name %CONTAINER% -p 8082:80 %IMAGE%:latest
                    ping -n 4 127.0.0.1 >NUL
                    curl -f http://localhost:8082/
                '''
            }
            post {
                always {
                    bat '"%DOCKER%" rm -f %CONTAINER% 2>NUL'
                }
            }
        }

        stage('4 - Docker Hub Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat '''
                        echo %DOCKER_PASS% | "%DOCKER%" login -u %DOCKER_USER% --password-stdin
                        "%DOCKER%" push %IMAGE%:build-%BUILD_NUMBER%
                        "%DOCKER%" push %IMAGE%:latest
                        "%DOCKER%" logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS : Build, Smoke Test et Push termines.'
        }
        failure {
            echo 'ECHEC : Verifier les logs Jenkins.'
        }
    }
}
