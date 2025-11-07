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
                // Fetch repo (if not using Jenkins SCM config)
                git branch: 'main', url: 'https://github.com/anildevops7702/docker-project.git'
                sh 'ls -l'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "🚀 Building Docker image..."
                    sh '''
                        docker build -t $IMAGE_NAME:${BUILD_NUMBER} .
                        docker tag $IMAGE_NAME:${BUILD_NUMBER} $IMAGE_NAME:latest
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
                            docker push $IMAGE_NAME:${BUILD_NUMBER}
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
                        echo "Updating image in deployment file..."
                        sed -i "s|image: .*|image: $IMAGE_NAME:${BUILD_NUMBER}|" $KUBERNETES_DEPLOYMENT

                        echo "Applying Kubernetes deployment..."
                        kubectl apply -f $KUBERNETES_DEPLOYMENT --validate=false

                        echo "✅ Waiting for rollout to complete..."
                        kubectl rollout status deployment/flask-app

                        echo "📦 Current pods:"
                        kubectl get pods -o wide

                        echo "🌐 Current services:"
                        kubectl get svc -o wide
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Deployment successful! Access your app at: http://$(minikube ip):30007"
        }
        failure {
            echo "❌ Deployment failed! Please check Jenkins logs."
        }
    }
}
