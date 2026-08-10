pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'kartikhiremath/weather_app:latest'
        SONARQUBE_ENV = 'MySonarQube' // must match your Jenkins SonarQube config
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Kartik-Hiremath/weather_app.git', credentialsId: 'dockerhub-creds'
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --exit-code 0 --severity LOW,MEDIUM,HIGH .'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONAR_TOKEN = credentials('sonarqube-token') // must be a Jenkins credential
            }
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=weather_app \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Trivy Docker Image Scan') {
            steps {
                sh 'trivy image $DOCKER_IMAGE'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $DOCKER_IMAGE
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
