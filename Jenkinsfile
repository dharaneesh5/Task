pipeline {
  agent any

  environment {
    DOCKER_USER = "dharaneesh5"
    TARGET_HOST = "54.254.165.108"
    TARGET_USER = "ubuntu"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build Images') {
      steps {
        sh '''
          docker build -t $DOCKER_USER/app1:latest app1
          docker build -t $DOCKER_USER/app2:latest app2
        '''
      }
    }

    stage('Push to DockerHub') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DH_USER',
          passwordVariable: 'DH_PASS'
        )]) {
          sh '''
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
            docker push $DOCKER_USER/app1:latest
            docker push $DOCKER_USER/app2:latest
          '''
        }
      }
    }

    stage('Deploy on EC2') {
      steps {
        sshagent(credentials: ['target-ssh']) {
          sh '''
            ssh -o StrictHostKeyChecking=no $TARGET_USER@$TARGET_HOST "
              docker pull $DOCKER_USER/app1:latest &&
              docker pull $DOCKER_USER/app2:latest &&

              docker stop app1 || true && docker rm app1 || true &&
              docker stop app2 || true && docker rm app2 || true &&

              docker run -d --restart unless-stopped --name app1 -p 5001:80 $DOCKER_USER/app1:latest &&
              docker run -d --restart unless-stopped --name app2 -p 5002:80 $DOCKER_USER/app2:latest
            "
          '''
        }
      }
    }
  }
}

