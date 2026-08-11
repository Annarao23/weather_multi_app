pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'annarao23/weather_app:latest'
        SONARQUBE_ENV = 'sonar-server'
        // Updated to your actual Docker Desktop installation directory
        PATH = "C:\\Users\\laxmi\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin;${env.PATH}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Annarao23/weather_multi_app.git'
            }
        }

        stage('Trivy FS Scan') {
            steps {
                bat 'trivy fs --exit-code 0 --severity LOW,MEDIUM,HIGH .'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONAR_TOKEN = credentials('sonarqube-token')
            }
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'
                    withSonarQubeEnv("${SONARQUBE_ENV}") {
                        bat """
                            "${scannerHome}\\bin\\sonar-scanner.bat" ^
                            -Dsonar.projectKey=weather_app ^
                            -Dsonar.sources=. ^
                            -Dsonar.host.url=%SONAR_HOST_URL% ^
                            -Dsonar.login=%SONAR_TOKEN%
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '"C:\\Users\\laxmi\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe" build -t %DOCKER_IMAGE% .'
            }
        }

        stage('Trivy Docker Image Scan') {
            steps {
                bat 'trivy image %DOCKER_IMAGE%'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    bat '''
                        "C:\\Users\\laxmi\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe" login -u %DOCKER_USER% -p %DOCKER_PASS%
                        "C:\\Users\\laxmi\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe" push %DOCKER_IMAGE%
                    '''
                }
            }
        }

        stage('Deployment Notification') {
            steps {
                echo '✅ App successfully deployed on Render at: https://weather-app-8k4q.onrender.com'
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        failure {
            echo '❌ Pipeline failed!'
        }
        success {
            echo '✅ Pipeline completed successfully!'
        }
    }
}
