pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        DOCKER = 'C:\\Users\\Famory Toure\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        IMAGE = 'famorytoure/monapp'
        CONTAINER = 'monapp-smoke'
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
                        credentialsId: 'dockerhub-push',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat '''
                        @echo off
                        setlocal

                        set "DOCKER_CONFIG=%WORKSPACE%\\.docker-auth"
                        if not exist "%DOCKER_CONFIG%" mkdir "%DOCKER_CONFIG%"

                        powershell -NoProfile -NonInteractive -Command "$env:DOCKER_PASS | & $env:DOCKER login -u $env:DOCKER_USER --password-stdin; exit $LASTEXITCODE"
                        if errorlevel 1 goto failed

                        powershell -NoProfile -NonInteractive -Command "for ($i = 1; $i -le 3; $i++) { & $env:DOCKER push ($env:IMAGE + ':build-' + $env:BUILD_NUMBER); if ($LASTEXITCODE -eq 0) { exit 0 }; if ($i -lt 3) { Start-Sleep -Seconds 10 } }; exit 1"
                        if errorlevel 1 goto failed

                        powershell -NoProfile -NonInteractive -Command "for ($i = 1; $i -le 3; $i++) { & $env:DOCKER push ($env:IMAGE + ':latest'); if ($LASTEXITCODE -eq 0) { exit 0 }; if ($i -lt 3) { Start-Sleep -Seconds 10 } }; exit 1"
                        if errorlevel 1 goto failed

                        "%DOCKER%" logout
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

        stage('5 - Deploy Docker Compose') {
            steps {
                bat '''
                    @echo off
                    "%DOCKER%" compose -f "%WORKSPACE%\\docker-compose.yml" pull
                    if errorlevel 1 exit /b 1

                    "%DOCKER%" compose -f "%WORKSPACE%\\docker-compose.yml" up -d
                    if errorlevel 1 exit /b 1
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS : Checkout, Build, Smoke Test, Push Docker Hub et Deploy Docker Compose termines.'
        }
        failure {
            echo 'ECHEC : Une etape du pipeline a echoue. Consulter les logs.'
        }
    }
}
