pipeline {
    agent any

    environment {
        PYTHON = "C:\\Users\\srivi.DESKTOP-L6OI7G9\\AppData\\Local\\Python\\bin\\python.exe"
        IMAGE_NAME = "srividya1008/aceest-service"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Pulling latest code from GitHub...'
                checkout scm
            }
        }

        stage('Environment Setup') {
            steps {
                echo 'Setting up Python virtual environment...'
                bat """
                    "${PYTHON}" -m venv venv
                    call venv\\Scripts\\activate.bat
                    venv\\Scripts\\python.exe -m pip install --upgrade pip
                    venv\\Scripts\\pip.exe install -r requirements.txt
                    venv\\Scripts\\pip.exe install flake8 pytest
                """
            }
        }

        stage('Lint') {
            steps {
                echo 'Running flake8 checks...'
                bat """
                    call venv\\Scripts\\activate.bat
                    venv\\Scripts\\python.exe -m flake8 app.py --select=E9,F63,F7,F82 --show-source --statistics
                """
            }
        }

        stage('Unit Tests') {
            steps {
                echo 'Running Pytest tests...'
                bat """
                    call venv\\Scripts\\activate.bat
                    venv\\Scripts\\python.exe -m pytest tests/ -v --tb=short
                """
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    bat """
                    C:\\sonar-scanner\\bin\\sonar-scanner.bat ^
                    -Dsonar.projectKey=aceest-fitness ^
                    -Dsonar.sources=. ^
                    -Dsonar.host.url=http://localhost:9000 ^
                    -Dsonar.login=%SONAR_AUTH_TOKEN%
                    """
                }
            }
        }
        
        stage('Docker Build and Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    bat """
                        docker login -u %DOCKER_USER% -p %DOCKER_PASS%
                        docker build -t %IMAGE_NAME%:%BUILD_NUMBER% .
                        docker push %IMAGE_NAME%:%BUILD_NUMBER%
                    """
                }
            }
        }
    }

    post {

        success {
            echo 'BUILD SUCCESS - ACEest pipeline completed successfully.'
        }

        failure {
            echo 'BUILD FAILED - Check logs.'
        }

        always {
            echo 'Cleaning up workspace...'
            bat 'if exist venv rmdir /s /q venv'
        }
    }
}
