pipeline {
  agent any

  environment {
    // Replace these with your real values:
    REPO_URL     = 'https://github.com/AmadeussSystem/flask-ci-cd.git'
    DOCKER_IMAGE = 'Esdeathhh/flask-app'
    DEPLOY_SERVER= 'ubuntu@13.232.21.132'   // ← your staging IP or hostname
  }

  stages {
    stage('Clone Repository') {
      steps {
        git url: REPO_URL, branch: 'main'  // or whatever branch you use
      }
    }

    stage('Build Docker Image') {
      steps {
        sh "docker build -t ${DOCKER_IMAGE}:latest ."
      }
    }

    stage('Push Docker Image') {
      steps {
        // Bind your Docker Hub creds (ID = 'docker-hub') to env vars
        withCredentials([usernamePassword(
            credentialsId: 'docker-hub',
            usernameVariable: 'DOCKER_USERNAME',
            passwordVariable: 'DOCKER_PASSWORD'
        )]) {
          sh """
            echo "\$DOCKER_PASSWORD" | docker login -u "\$DOCKER_USERNAME" --password-stdin
            docker push ${DOCKER_IMAGE}:latest
          """
        }
      }
    }

    stage('Deploy to Staging') {
      steps {
        // Uses your SSH key credential (ID = 'deploy-key')
        sshagent(['deploy-key']) {
          sh """
            ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER} \\
              "docker pull ${DOCKER_IMAGE}:latest && \\
               docker stop flask-app || true && \\
               docker rm flask-app  || true && \\
               docker run -d --name flask-app -p 5000:5000 ${DOCKER_IMAGE}:latest"
          """
        }
      }
    }
  }

  post {
    always {
      echo '✅ Pipeline finished'
    }
  }
}
