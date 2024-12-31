pipeline {
    agent {
        docker { image 'docker:24.0.1' }
    }

    environment {
        GCP_PROJECT_ID = 'belajar-terraform-dan-ansible'
        IMAGE_NAME = 'nodejs-app'
        MIG_SINGAPORE = 'asia-sg-mig'
        MIG_JAKARTA = 'asia-jkt-mig'
        REGION_SINGAPORE = 'asia-southeast1'
        REGION_JAKARTA = 'asia-southeast2'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install Docker Tools') {
            steps {
                script {
                    sh """
                        apk update && apk add --no-cache curl
                        curl -fsSL https://get.docker.com | sh
                        docker --version
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t gcr.io/${GCP_PROJECT_ID}/${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Push Docker Image to GCP Container Registry') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh "gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}"
                        sh "gcloud auth configure-docker"
                        sh "docker push gcr.io/${GCP_PROJECT_ID}/${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('Update MIGs') {
            parallel {
                stage('Update Singapore MIG') {
                    steps {
                        script {
                            withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                                sh """
                                    gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                                    gcloud compute instance-groups managed rolling-action start-update ${MIG_SINGAPORE} \
                                        --region=${REGION_SINGAPORE} \
                                        --version=template=gcr.io/${GCP_PROJECT_ID}/${IMAGE_NAME}:latest
                                """
                            }
                        }
                    }
                }

                stage('Update Jakarta MIG') {
                    steps {
                        script {
                            withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                                sh """
                                    gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                                    gcloud compute instance-groups managed rolling-action start-update ${MIG_JAKARTA} \
                                        --region=${REGION_JAKARTA} \
                                        --version=template=gcr.io/${GCP_PROJECT_ID}/${IMAGE_NAME}:latest
                                """
                            }
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment to MIGs completed successfully!'
        }
        failure {
            echo 'Deployment failed. Please check the logs.'
        }
    }
}
