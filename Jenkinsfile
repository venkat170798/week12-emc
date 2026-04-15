pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "venkat170798/sample-app"
    }

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/venkat170798/week12-emc.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'pip install -r requirements.txt'
                sh 'pip install pytest'
                sh 'pytest'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=sample-app \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://<your-ip>:9000 \
                    -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                docker stop sample-app || true
                docker rm sample-app || true
                docker run -d -p 5000:5000 --name sample-app $DOCKER_IMAGE
                '''
            }
        }
    }
}
