pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        DOCKERHUB_USER = 'your-dockerhub-username'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/yourusername/devops-pipeline-sample.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh 'docker build -t $DOCKERHUB_USER/backend:latest ./backend'
                    sh 'docker build -t $DOCKERHUB_USER/frontend:latest ./frontend'
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_USER --password-stdin'
                    sh 'docker push $DOCKERHUB_USER/backend:latest'
                    sh 'docker push $DOCKERHUB_USER/frontend:latest'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'kubectl apply -f k8s/'
                }
            }
        }
    }
}
