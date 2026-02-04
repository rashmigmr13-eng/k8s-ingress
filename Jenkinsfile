pipeline {
    agent any

    environment {
        IMAGE_NAME = "rashmidevops1/test-dev"
        IMAGE_TAG  = "7"
        DEPLOY_FILE = "deploy.yaml"
    }

    stages {

        stage('Checkout Jenkinsfile Repo') {
            steps {
                git url: 'https://github.com/rashmigmr13-eng/k8s-ingress.git', branch: 'master'
            }
        }

        stage('Clone App Repo') {
            steps {
                git url: 'https://github.com/manjukolkar/scroll-web.git', branch: 'master'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-password',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Image') {
            steps {
                sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // Replace IMAGE_PLACEHOLDER in deploy.yaml with the new image
                sh "sed -i \"s|IMAGE_PLACEHOLDER|${IMAGE_NAME}:${IMAGE_TAG}|g\" ${DEPLOY_FILE}"
                sh "microk8s kubectl apply -f ${DEPLOY_FILE}"
                sh "microk8s kubectl rollout status deployment/my-deploy-app"
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "Waiting 10 seconds for service to be reachable..."
                sh "sleep 10"
                echo "Checking application URL..."
                sh "curl -f http://3.9.172.47:30080"
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline succeeded"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
