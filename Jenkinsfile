pipeline {
  agent any

  environment {
    IMAGE_NAME = 'kushagrag99/monitoring-app'
    REGISTRY_CREDENTIALS = 'docker-cred'          // DockerHub credentials in Jenkins
    KUBECONFIG_CREDENTIAL_ID = 'kubeconfig-jenkins'  // Kubernetes kubeconfig credentials in Jenkins
    PATH = "${env.PATH}:/usr/local/bin"
  }

  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/kushagra-g9/monitoring-node-app.git', branch: 'main'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          IMAGE_TAG = "${IMAGE_NAME}:${env.BUILD_NUMBER}"
          sh "docker build -t $IMAGE_TAG ."
        }
      }
    }

    stage('Push to DockerHub') {
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', REGISTRY_CREDENTIALS) {
            sh "docker push $IMAGE_TAG"
          }
        }
      }
    }

   stage('Deploy to Kubernetes') {
  steps {
    script {
      IMAGE_TAG = "${IMAGE_NAME}:${env.BUILD_NUMBER}"
    }
    withCredentials([file(credentialsId: KUBECONFIG_CREDENTIAL_ID, variable: 'KUBECONFIG_FILE')]) {
      sh """
        export KUBECONFIG=$KUBECONFIG_FILE
        sed -i 's|kushagrag99/monitoring-app:latest|${IMAGE_TAG}|g' k8s/deployment.yaml
        kubectl apply -f k8s/
      """
    }
  }
}

  }

  post {
    success {
      echo "✅ Deployment successful: $IMAGE_TAG"
    }
    failure {
      echo "❌ Build or deployment failed."
    }
  }
}
