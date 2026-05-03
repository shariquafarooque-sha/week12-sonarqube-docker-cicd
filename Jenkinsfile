pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "shariquafarooque/week12-cicd-app"
        CONTAINER_NAME = "week12-cicd-container"
        SONARQUBE_ENV = "SonarQube"
    }

    stages {
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
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:latest .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat '''
                    echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                    docker push %DOCKER_IMAGE%:latest
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