pipeline {
    agent any
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['UAT','PROD'], description: 'Select deployment environment')
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-id') // Jenkins credential ID
        AWS_CREDENTIALS = credentials('aws-ssh-key-id')      // Jenkins SSH key credential ID
        IMAGE_NAME = 'swatibalodi01/dotnet-hello-world'    //dockerhub repo name
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'git@github.com:swatibalodi/dotnet-core-jenkins-project.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${params.ENVIRONMENT}")
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', "${DOCKERHUB_CREDENTIALS}") {
                        docker.image("${IMAGE_NAME}:${params.ENVIRONMENT}").push()
                    }
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                script {
                    def EC2_IP = (params.ENVIRONMENT == 'UAT') ? 'UAT_EC2_IP' : 'PROD_EC2_IP'
                    sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${EC2_IP} '
                        docker stop dotnet-test || true
                        docker rm dotnet-test || true
                        docker pull ${IMAGE_NAME}:${params.ENVIRONMENT}
                        docker run -d -p 5000:80 --name dotnet-app ${IMAGE_NAME}:${params.ENVIRONMENT}'
                    """
                }
            }
        }
        stage('Health Check') {
            steps {
                script {
                    def EC2_IP = (params.ENVIRONMENT == 'UAT') ? 'UAT_EC2_IP' : 'PROD_EC2_IP'
                    sh "curl http://${EC2_IP}/api/hello"
                }
            }
        }
    }
}
