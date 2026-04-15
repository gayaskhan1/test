pipeline {
    agent any

    environment {
        IMAGE_NAME = "test-app"
        DOCKERHUB_CREDENTIALS = "dockerhub-creds"
        DOCKERHUB_USERNAME = "gayaskhan1"
        SONARQUBE_SERVER = "sonar"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/gayaskhan1/test.git'
            }
        }

        stage('Build Application') {
            steps {
                sh 'npm install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=test-app \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://sonarqube:9000 \
                    -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t test-app .'
            }
        }

        stage('Trivy Security Scan') {
            steps {
                sh '''
                docker run --rm \
                -v /var/run/docker.sock:/var/run/docker.sock \
                aquasec/trivy image test-app
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS')]) {

                    sh '''
                    docker login -u $USER -p $PASS
                    docker tag test-app gayaskhan1/test-app:latest
                    docker push gayaskhan1/test-app:latest
                    '''
                }
            }
        }
    }
}
