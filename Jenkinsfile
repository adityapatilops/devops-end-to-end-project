pipeline {
    agent any

    environment {
        IMAGE_NAME = "adityapatilops/flask-app"
        IMAGE_TAG = "v2"
        EC2_HOST = "16.192.62.91"
        EC2_USER = "ubuntu"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/adityapatilops/devops-end-to-end-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %IMAGE_NAME%:%IMAGE_TAG% .'
            }
        }

        stage('Docker Login Test') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    bat '''
                    echo USER=%DOCKER_USER%
                    docker logout
                    echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat 'docker push %IMAGE_NAME%:%IMAGE_TAG%'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh']) {
                    bat """
                    ssh -o StrictHostKeyChecking=no %EC2_USER%@%EC2_HOST% ^
                    "docker pull %IMAGE_NAME%:%IMAGE_TAG% && ^
                    docker stop flask-container || true && ^
                    docker rm flask-container || true && ^
                    docker run -d --name flask-container -p 5000:5000 %IMAGE_NAME%:%IMAGE_TAG%"
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful!'
        }

        failure {
            echo 'Deployment Failed!'
        }
    }
}