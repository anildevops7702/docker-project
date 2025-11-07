pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'
        IMAGE_NAME = 'thannirudocker/flask-ecommerce'
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/anildevops7702/docker-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "🐳 Building Docker image..."
                    sh '''
                        docker build -t $IMAGE_NAME:$BUILD_NUMBER .
                    '''
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    echo "📤 Pushing image to Docker Hub..."
                    withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS,
                                                      usernameVariable: 'DOCKER_USER',
                                                      passwordVariable: 'DOCKER_PASS')]) {
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push $IMAGE_NAME:$BUILD_NUMBER
                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    echo "⚙️ Deploying to Minikube..."
                    sh '''
                        export KUBECONFIG=$KUBECONFIG
                        echo "Updating image in deployment file..."
                        sed -i "s|image: .*|image: $IMAGE_NAME:$BUILD_NUMBER|" deployment.yaml

                        echo "Applying Kubernetes deployment..."
                        kubectl apply -f deployment.yaml --validate=false

                        echo "Waiting for pods to be ready..."
                        kubectl rollout status deployment/flask-app --timeout=90s
                        echo "🎉 Deployment successful! Access your app at: http://$(minikube ip):30007"
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo "❌ Deployment failed! Please check Jenkins logs."
        }
    }
}
