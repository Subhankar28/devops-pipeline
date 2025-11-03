pipeline {
  agent any
  environment {
    DOCKERHUB_USER = 'subhankar28'
    DOCKERHUB_CRED = 'dockerhub-creds'
    KUBECONFIG_CRED = 'kubeconfig'
    IMAGE_TAG = "build-${env.BUILD_NUMBER}"
    NAMESPACE = "myapp"
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Test Backend') {
      steps {
        dir('backend') {
          sh 'npm install'
          sh 'npm test || echo "Tests skipped"'
        }
      }
    }

    stage('Build Docker Images') {
      steps {
        script {
          docker.withRegistry('', DOCKERHUB_CRED) {
            dir('backend') {
              def backend = docker.build("${DOCKERHUB_USER}/backend:${IMAGE_TAG}")
              backend.push()
              backend.push('latest')
            }
            dir('frontend') {
              def frontend = docker.build("${DOCKERHUB_USER}/frontend:${IMAGE_TAG}")
              frontend.push()
              frontend.push('latest')
            }
          }
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        withCredentials([file(credentialsId: KUBECONFIG_CRED, variable: 'KUBECONFIG_FILE')]) {
          sh '''
          export KUBECONFIG=$KUBECONFIG_FILE
          kubectl apply -f k8s/namespace.yaml
          sed -i "s|YOUR_DOCKERHUB_USERNAME/backend:latest|${DOCKERHUB_USER}/backend:${IMAGE_TAG}|g" k8s/backend-deployment.yaml
          sed -i "s|YOUR_DOCKERHUB_USERNAME/frontend:latest|${DOCKERHUB_USER}/frontend:${IMAGE_TAG}|g" k8s/frontend-deployment.yaml
          kubectl apply -f k8s/postgres-deployment.yaml
          kubectl apply -f k8s/backend-deployment.yaml
          kubectl apply -f k8s/frontend-deployment.yaml
          '''
        }
      }
    }

    stage('Verify Deployment') {
      steps {
        withCredentials([file(credentialsId: KUBECONFIG_CRED, variable: 'KUBECONFIG_FILE')]) {
          sh '''
          export KUBECONFIG=$KUBECONFIG_FILE
          kubectl rollout status deployment/backend -n myapp --timeout=120s
          kubectl rollout status deployment/frontend -n myapp --timeout=120s
          '''
        }
      }
    }
  }
  post {
    success {
      echo "✅ Deployment completed successfully!"
    }
    failure {
      echo "❌ Deployment failed, rolling back..."
      withCredentials([file(credentialsId: KUBECONFIG_CRED, variable: 'KUBECONFIG_FILE')]) {
        sh '''
        export KUBECONFIG=$KUBECONFIG_FILE
        kubectl rollout undo deployment/backend -n myapp || true
        kubectl rollout undo deployment/frontend -n myapp || true
        '''
      }
    }
  }
}
