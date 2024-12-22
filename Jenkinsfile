pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'nodejs-app' // Mendeklarasikan nama image Docker
    }

    stages {
        stage('Clone') {
            steps {
                script {
                    // Menarik kode dari repositori GitHub
                    git 'https://github.com/Fitriawan-Arya-N/nodejs-app.git'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Membangun Docker image
                    docker.build("${DOCKER_IMAGE}")
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    // Menjalankan container Docker yang telah dibangun
                    docker.image("${DOCKER_IMAGE}").run('-d -p 3000:3000')
                }
            }
        }
    }

    post {
        always {
            // Membersihkan resource Docker setelah pipeline selesai
            echo 'Cleaning up Docker images and containers...'
            sh 'docker ps -aq | xargs docker stop || true'
            sh 'docker ps -aq | xargs docker rm || true'
        }
    }
}
