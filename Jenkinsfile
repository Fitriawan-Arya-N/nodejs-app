pipeline {
    agent any
    stages {
        stage('Clone') {
            steps {
                git 'https://github.com/Fitriawan-Arya-N/nodejs-app.git'
            }
        }
    }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build('nodejs-app')
                }
            }
        }
        stage('Run Docker Container') {
            steps {
                script {
                    docker.image('nodejs-app').run('-d -p 3000:3000')
                }
            }
        }    
}