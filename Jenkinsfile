
pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'
        DOCKERHUB_USERNAME = 'YOUR_DOCKERHUB_USERNAME'
        IMAGE_NAME = 'student-k8s-demo'
        K8S_DEPLOYMENT = 'student-k8s-demo'
        K8S_CONTAINER = 'student-k8s-demo'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                bat '''
                    python -m pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                bat 'python -m pytest -v'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKERHUB_USERNAME%/%IMAGE_NAME%:%BUILD_NUMBER% .'
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', DOCKERHUB_CREDENTIALS) {
                        def image = docker.image(
                            "${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}"
                        )
                        image.push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat """
                    kubectl set image deployment/%K8S_DEPLOYMENT% %K8S_CONTAINER%=%DOCKERHUB_USERNAME%/%IMAGE_NAME%:%BUILD_NUMBER%
                    kubectl rollout status deployment/%K8S_DEPLOYMENT%
                """
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the stage logs.'
        }
    }
}
