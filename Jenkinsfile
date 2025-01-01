pipeline {
    agent any

    environment {
        GCP_PROJECT_ID = 'belajar-terraform-dan-ansible'
        IMAGE_NAME = 'nodejs-app'
        MIG_SINGAPORE = 'asia-sg-mig'
        MIG_JAKARTA = 'asia-jkt-mig'
        REGION_SINGAPORE = 'asia-southeast1'
        REGION_JAKARTA = 'asia-southeast2'
        REPOSITORY_NAME = 'nodejs-docker-image-repo'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Authenticate with GCP') {
            steps {
                withCredentials([file(credentialsId: 'gcp-jenkins-vm', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    script {
                        // Authenticate with Google Cloud using the service account credentials
                        sh "gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}"
                        sh "gcloud config set project ${GCP_PROJECT_ID}"
                        sh "gcloud auth configure-docker"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t gcr.io/${GCP_PROJECT_ID}/${REPOSITORY_NAME}/${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Push Docker Image to GCP Artifact Registry') {
            steps {
                script {
                    sh "docker push gcr.io/${GCP_PROJECT_ID}/${REPOSITORY_NAME}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Update MIGs') {
            parallel {
                stage('Update Singapore MIG') {
                    steps {
                        script {
                            sh """
                                gcloud compute instance-groups managed rolling-action start-update ${MIG_SINGAPORE} \
                                    --region=${REGION_SINGAPORE} \
                                    --version=template=gcr.io/${GCP_PROJECT_ID}/${REPOSITORY_NAME}/${IMAGE_NAME}:latest
                            """
                        }
                    }
                }

                stage('Update Jakarta MIG') {
                    steps {
                        script {
                            sh """
                                gcloud compute instance-groups managed rolling-action start-update ${MIG_JAKARTA} \
                                    --region=${REGION_JAKARTA} \
                                    --version=template=gcr.io/${GCP_PROJECT_ID}/${REPOSITORY_NAME}/${IMAGE_NAME}:latest
                            """
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
    }
}
