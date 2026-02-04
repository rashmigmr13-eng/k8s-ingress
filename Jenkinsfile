pipeline {
    agent any
    environment {
        DOCKER_USER = 'rashmidevops1'
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
                sh 'docker build -t rashmidevops1/test-dev:7 .'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-password', 
                                                  usernameVariable: 'DOCKER_USER', 
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Image') {
            steps {
                sh 'docker push rashmidevops1/test-dev:7'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                   sed -i s|IMAGE_PLACEHOLDER|rashmidevops1/test-dev:7|g deploy.yaml
                   microk8s kubectl apply -f deploy.yaml
                   microk8s kubectl rollout status deployment/my-deploy-app
                   '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Waiting 10 seconds for service to be reachable...'
                sh 'sleep 10'
                echo 'Checking application URL...'
                sh 'curl -f http://3.9.172.47:30080'
            }
        }
    }

    post {
        failure {
            echo '❌ Pipeline failed'
        }
    }
}
