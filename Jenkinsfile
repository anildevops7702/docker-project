pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'
        IMAGE_NAME = 'thannirudocker/flask-ecommerce'
        KUBERNETES_DEPLOYMENT = 'deployment.yaml'
    }

    stages {

        stage('Checkout') {
            steps {
                echo "✅ Checking out source code from GitHub..."
                // Jenkins automatically checks out your repo based on SCM config
                sh 'git branch'
                sh 'ls -l'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "🚀 Building Docker image..."
                    sh '''
                        docker build -t $IMAGE_NAME:latest .
                        docker images | grep flask-ecommerce
                    '''
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    echo "📤 Pushing image to DockerHub..."
                    withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push $IMAGE_NAME:latest
                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes (Minikube)') {
            steps {
                script {
                    echo "⚙️ Deploying to Minikube..."
                    sh '''
                        echo "Applying Kubernetes deployment..."
                        kubectl apply -f $KUBERNETES_DEPLOYMENT

                        echo "✅ Waiting for rollout to complete..."
                        kubectl rollout status deployment/flask-app

                        echo "📦 Current pods:"
                        kubectl get pods

                        echo "🌐 Current services:"
                        kubectl get svc
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Deployment successful!"
        }
        failure {
            echo "❌ Deployment failed! Please check Jenkins logs."
        }
    }
}
