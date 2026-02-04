pipeline {
    agent any

    environment {
        DOCKER_USER = 'rashmidevops1'
        DOCKER_IMAGE = 'rashmidevops1/test-dev:7'
        DOCKER_PASSWORD = credentials('docker-hub-password') // Use Jenkins credentials
        KUBE_URL = 'http://3.9.172.47:30080'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Clone App Repo') {
            steps {
                git url: 'https://github.com/manjukolkar/scroll-web.git', branch: 'master'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([string(credentialsId: 'docker-hub-password', variable: 'PASS')]) {
                    sh "echo $PASS | docker login -u ${DOCKER_USER} --password-stdin"
                }
            }
        }

        stage('Push Image') {
            steps {
                sh "docker push ${DOCKER_IMAGE}"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                sed -i s|IMAGE_PLACEHOLDER|${DOCKER_IMAGE}|g deploy.yaml
                microk8s kubectl apply -f deploy.yaml
                microk8s kubectl rollout status deployment/my-deploy-app
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Waiting 10 seconds for service to be reachable...'
                sh 'sleep 10'
                echo 'Checking application URL...'
                sh "curl -f ${KUBE_URL}"
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline succeeded'
        }
        failure {
            echo '❌ Pipeline failed'
        }
    }
}
