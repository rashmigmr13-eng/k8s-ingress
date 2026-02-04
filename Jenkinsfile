pipeline {
    agent any

    environment {
        IMAGE = "rashmidevops1/test-dev"
        TAG = "${BUILD_NUMBER}"
        DOMAIN = "3.9.172.47"   // Your public IP
        NODEPORT = "30080"       // NodePort for external access
    }

    stages {

        stage('Clone App Repo') {
            steps {
                git url: 'https://github.com/manjukolkar/scroll-web.git', branch: 'master'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $IMAGE:$TAG ."
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'U', passwordVariable: 'P')]) {
                    sh 'echo $P | docker login -u $U --password-stdin'
                }
            }
        }

        stage('Push Image') {
            steps {
                sh "docker push $IMAGE:$TAG"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                # Replace placeholder image with the new build
                sed -i 's|IMAGE_PLACEHOLDER|$IMAGE:$TAG|g' deploy.yaml

                # Apply deployment, service, ingress
                microk8s kubectl apply -f deploy.yaml

                # Wait for deployment to complete
                microk8s kubectl rollout status deployment/my-deploy-app
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "Waiting 10 seconds for service to be reachable..."
                sh "sleep 10"

                echo "Checking application URL..."
                sh "curl -f http://$DOMAIN:$NODEPORT || exit 1"
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful! Visit: http://$DOMAIN:$NODEPORT"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
