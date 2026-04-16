pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "venkatr271/sample-app"
        
    }

    stages {



       stage('Build & Test') {
    steps {
        sh '''
        python3 -m venv venv
        . venv/bin/activate
        pip install -r requirements.txt
        pip install pytest
        pytest
        '''
    }
}

stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('SonarQube') {
            script {
                def scannerHome = tool 'SonarScanner'
                sh """
                ${scannerHome}/bin/sonar-scanner \
                -Dsonar.projectKey=sample-app \
                -Dsonar.sources=. \
                -Dsonar.host.url=http://3.110.154.249:9000 \
                -Dsonar.exclusions=venv/**
                """
            }
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
