pipeline {
    agent any
    environment {
        REPO_URL     = 'https://github.com/<your-username>/flask-ci-cd.git'
        DOCKER_IMAGE = 'yourdockerhub/flask-app'
        DEPLOY_SERVER= 'ec2-user@<staging-ip>'
    }
    stages {
        stage('Clone Repository') {
            steps {
                git REPO_URL
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:latest .'
            }
        }
        stage('Push Docker Image') {
            steps {
                sh '''
                  echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                  docker push ${DOCKER_IMAGE}:latest
                '''
            }
        }
        stage('Deploy to Staging') {
            steps {
                sshagent(['deploy-key']) {
                    sh '''
                      ssh ${DEPLOY_SERVER} \
                      "docker pull ${DOCKER_IMAGE}:latest && \
                       docker run -d -p 5000:5000 ${DOCKER_IMAGE}:latest"
                    '''
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline execution completed.'
        }
    }
}
