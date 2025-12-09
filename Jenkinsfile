pipeline {
  agent any
  options { timestamps() }

  environment {
    IMAGE = "TON_DOCKER_USERNAME/monapp"
    TAG   = "build-${env.BUILD_NUMBER}"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Docker Build') {
      steps {
        sh "docker build -t ${IMAGE}:${TAG} ."
      }
    }

    stage('Smoke Test') {
      steps {
        sh """
          docker rm -f monapp_test || true
          docker run -d --name monapp_test -p 8090:80 ${IMAGE}:${TAG}
          sleep 5
          curl -I http://localhost:8090 | grep '200 OK'
          docker rm -f monapp_test
        """
      }
    }

    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh """
            echo "$PASS" | docker login -u "$USER" --password-stdin
            docker tag ${IMAGE}:${TAG} ${IMAGE}:latest
            docker push ${IMAGE}:${TAG}
            docker push ${IMAGE}:latest
          """
        }
      }
    }
  }

  post {
    success {
      echo "🎉 Pipeline Docker COMPLET : Build + Test + Push OK !"
    }
    failure {
      echo "❌ Erreur dans le pipeline"
    }
  }
}
