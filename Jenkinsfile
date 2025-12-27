pipeline {
    agent any

    environment {
        IMAGE = "dharshankumarj/openwebui"
        TAG = "latest"
        VM_USER = "azureuser"
        VM_IP = "74.225.196.70"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                sh """
                docker build -t $IMAGE:$TAG .
                """
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                }
            }
        }

        stage('Push Image') {
            steps {
                sh "docker push $IMAGE:$TAG"
            }
        }

        stage('Deploy to Azure VM') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no $VM_USER@$VM_IP \
                '/home/azureuser/deploy_openwebui.sh'
                """
            }
        }
    }

    post {
        always {
            sh 'docker system prune -af'
        }
        success {
            echo "CI + CD completed successfully"
            sh "docker system prune -af"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
