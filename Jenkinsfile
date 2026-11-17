
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
                    @echo off
                    "%DOCKER%" build -t %IMAGE%:build-%BUILD_NUMBER% .
                    if errorlevel 1 exit /b 1

                    "%DOCKER%" tag %IMAGE%:build-%BUILD_NUMBER% %IMAGE%:latest
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('3 - Smoke Test') {
            steps {
                bat '''
                    @echo off
                    "%DOCKER%" rm -f %CONTAINER% 2>NUL
                    "%DOCKER%" run -d --name %CONTAINER% -p 8082:80 %IMAGE%:latest
                    if errorlevel 1 exit /b 1

                    ping -n 4 127.0.0.1 >NUL

                    curl -f -s -o NUL -w "HTTP %%{http_code}" http://localhost:8082/
                    if errorlevel 1 exit /b 1
                '''
            }
            post {
                always {
                    bat '''
                        @echo off
                        "%DOCKER%" rm -f %CONTAINER% 2>NUL
                        exit /b 0
                    '''
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
                        @echo off
                        setlocal

                        set "DOCKER_CONFIG=%WORKSPACE%\\.docker-auth"
                        if not exist "%DOCKER_CONFIG%" mkdir "%DOCKER_CONFIG%"

                        echo %DOCKER_PASS% | "%DOCKER%" login -u %DOCKER_USER% --password-stdin
                        if errorlevel 1 goto failed

                        "%DOCKER%" push %IMAGE%:build-%BUILD_NUMBER%
                        if errorlevel 1 goto failed

                        "%DOCKER%" push %IMAGE%:latest
                        if errorlevel 1 goto failed

                        "%DOCKER%" logout
                        if errorlevel 1 goto failed

                        if exist "%DOCKER_CONFIG%" rmdir /s /q "%DOCKER_CONFIG%"
                        exit /b 0

                        :failed
                        "%DOCKER%" logout
                        if exist "%DOCKER_CONFIG%" rmdir /s /q "%DOCKER_CONFIG%"
                        exit /b 1
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS : Build, Smoke Test et Push Docker Hub termines.'
        }
        failure {
            echo 'ECHEC : Une etape du pipeline a echoue. Consulter les logs.'
        }
    }
}
