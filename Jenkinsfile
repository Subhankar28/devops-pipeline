pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        DOCKERHUB_USER = 'subhankar28'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Subhankar28/devops-pipeline.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    echo "🔧 Building backend and frontend Docker images..."
                    sh 'docker build -t $DOCKERHUB_USER/backend:latest ./backend'
                    sh 'docker build -t $DOCKERHUB_USER/frontend:latest ./frontend'
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    echo "📦 Pushing Docker images to DockerHub..."
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_USER --password-stdin'
                    sh 'docker push $DOCKERHUB_USER/backend:latest'
                    sh 'docker push $DOCKERHUB_USER/frontend:latest'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "🚀 Deploying to Kubernetes..."
                    sh 'kubectl apply -f k8s/'
                }
            }
        }
    }

    post {
        failure {
            echo "❌ Deployment failed, rolling back..."
            script {
                sh 'kubectl rollout undo deployment/backend || true'
                sh 'kubectl rollout undo deployment/frontend || true'
            }
        }
    }
}
