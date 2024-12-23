pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'fitriawanaryan/nodejs-app:latest'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Fitriawan-Arya-N/nodejs-app.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}", ".")
                }
            }
        }
        stage('Push to Docker Registry') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        docker.image("${DOCKER_IMAGE}").push()
                    }
                }
            }
        }
        stage('Deploy Application') {
            steps {
                script {
                    sh '''
                    docker stop nodejs-app || true
                    docker rm nodejs-app || true
                    docker run -d --name nodejs-app -p 3000:3000 ${DOCKER_IMAGE}
                    '''
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline Completed'
        }
        success {
            echo 'Application Deployed Successfully!'
        }
        failure {
            echo 'Pipeline Failed!'
        }
    }
}
