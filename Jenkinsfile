pipeline {
    agent any

    environment {
        // Change 'annarao23' to your actual Docker Hub username
        DOCKER_IMAGE = 'annarao23/weather_app:latest' 
        
        // Must match the Server Name in Manage Jenkins -> System -> SonarQube servers
        SONARQUBE_ENV = 'sonar-server' 
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
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    bat '''
                        sonar-scanner ^
                        -Dsonar.projectKey=weather_app ^
                        -Dsonar.sources=. ^
                        -Dsonar.host.url=%SONAR_HOST_URL% ^
                        -Dsonar.login=%SONAR_TOKEN%
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE% .'
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
                        echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                        docker push %DOCKER_IMAGE%
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
