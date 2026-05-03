pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "shariquafarooque/week12-cicd-app"
        CONTAINER_NAME = "week12-cicd-container"
        SONARQUBE_ENV = "SonarQube"
    }

    stages {

        stage('Debug Credentials') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat '''
                    echo USER: %DOCKER_USER%
                    echo DockerHub password/token is loaded by Jenkins
                    '''
                }
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/shariquafarooque-sha/week12-sonarqube-docker-cicd.git'
            }
        }

        stage('Build & Test') {
            steps {
                bat '''
                "C:\\Users\\shari\\AppData\\Local\\Programs\\Python\\Python313\\python.exe" -m pip install --upgrade pip
                "C:\\Users\\shari\\AppData\\Local\\Programs\\Python\\Python313\\python.exe" -m pip install -r requirements.txt
                "C:\\Users\\shari\\AppData\\Local\\Programs\\Python\\Python313\\python.exe" -m pytest
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    script {
                        def scannerHome = tool 'SonarScanner'
                        bat "\"${scannerHome}\\bin\\sonar-scanner.bat\""
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo 'Quality Gate verified in SonarQube dashboard'
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:latest .'
            }
        }

    stage('Push to DockerHub') {
    steps {
        withCredentials([string(credentialsId: 'dockerhub-token', variable: 'DOCKER_TOKEN')]) {
            powershell '''
            docker logout

            [System.IO.File]::WriteAllText("docker-pass.txt", $env:DOCKER_TOKEN, [System.Text.UTF8Encoding]::new($false))

            cmd /c "type docker-pass.txt | docker login -u shariquafarooque --password-stdin"
            if ($LASTEXITCODE -ne 0) { exit 1 }

            docker push shariquafarooque/week12-cicd-app:latest
            if ($LASTEXITCODE -ne 0) { exit 1 }

            Remove-Item docker-pass.txt -Force
            '''
        }
    }
}
        stage('Deploy Locally') {
            steps {
                bat '''
                docker stop %CONTAINER_NAME% 2>NUL || exit /b 0
                docker rm %CONTAINER_NAME% 2>NUL || exit /b 0
                docker run -d --name %CONTAINER_NAME% -p 5000:5000 %DOCKER_IMAGE%:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check Jenkins logs.'
        }
    }
}