pipeline {
    agent any

    environment {
        // Jenkins credentials
        DOCKERHUB = credentials('dockerhub-id')
        IMAGE_NAME = "swatibalodi01/dotnet-hello-world"
        IMAGE_TAG = "${BUILD_NUMBER}"

        // UAT server
        UAT_SERVER = "34.229.134.21"
        SSH_CREDENTIALS = "aws-ssh-key-id"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'master',
                url: 'https://github.com/swatibalodi/dotnet-core-jenkins-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh """
                echo ${DOCKERHUB_PSW} | docker login -u ${DOCKERHUB_USR} --password-stdin
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                docker push ${IMAGE_NAME}:${IMAGE_TAG}
                docker push ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Deploy to UAT EC2') {
            steps {
                sshagent(['aws-ssh-key-id']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@${UAT_SERVER} '
                    docker pull ${IMAGE_NAME}:latest
                    docker stop dotnet-app || true
                    docker rm dotnet-app || true
                    docker run -d -p 80:80 --name dotnet-app ${IMAGE_NAME}:latest
                    '
                    """
                }
            }
        }

        stage('Health Check') {
            steps {
                sh "curl -f http://${UAT_SERVER}/ || exit 1"
            }
        }
    }
}
