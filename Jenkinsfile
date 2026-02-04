pipeline {
    agent any

    environment {
        IMAGE = "rashmidevops1/test-dev"
        TAG = "${BUILD_NUMBER}"
        DOMAIN = "203.0.113.25"   // Replace with your MicroK8s server public IP
        NODEPORT = "30080"
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
                sed -i 's|IMAGE_PLACEHOLDER|$IMAGE:$TAG|g' deploy.yaml
                microk8s kubectl apply -f deploy.yaml
                microk8s kubectl rollout status deployment/my-deploy-app
                """
            }
        }

        stage('Verify') {
            steps {
                sh """
                sleep 10
                curl http://$DOMAIN:$NODEPORT
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
