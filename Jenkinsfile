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

        stage('Debug PATH and Permissions') {
            steps {
                sh """
                    echo \$PATH
                    ls -l /opt/google-cloud-sdk/bin/gcloud
                    /opt/google-cloud-sdk/bin/gcloud --version
                """
            }
        }

        stage('Authenticate with GCP') {
            steps {
                withCredentials([file(credentialsId: 'gcp-jenkins-vm', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    script {
                        sh """
                            export PATH=/opt/google-cloud-sdk/bin:\$PATH
                            gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                            gcloud config set project ${GCP_PROJECT_ID}
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker rmi -f asia-southeast2-docker.pkg.dev/${GCP_PROJECT_ID}/${REPOSITORY_NAME}/${IMAGE_NAME}:latest || true
                        docker build -t asia-southeast2-docker.pkg.dev/${GCP_PROJECT_ID}/${REPOSITORY_NAME}/${IMAGE_NAME}:latest .
                    """
                }
            }
        }

        stage('Push Docker Image to GCP Artifact Registry') {
            steps {
                script {
                    sh """
                        export PATH=/opt/google-cloud-sdk/bin:\$PATH
                        gcloud auth configure-docker asia-southeast2-docker.pkg.dev
                        docker push asia-southeast2-docker.pkg.dev/${GCP_PROJECT_ID}/${REPOSITORY_NAME}/${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Deploy to MIG in Singapore and Jakarta') {
            steps {
                script {
                    // Update Singapore MIG with the existing image and existing instance template
                    sh """
                        gcloud compute instance-groups managed rolling-action start-update ${MIG_SINGAPORE} \
                            --image asia-southeast2-docker.pkg.dev/${GCP_PROJECT_ID}/${REPOSITORY_NAME}/${IMAGE_NAME}:latest \
                            --image-project ${GCP_PROJECT_ID} \
                            --region ${REGION_SINGAPORE} \
                            --mode=single
                    """

                    // Update Jakarta MIG with the existing image and existing instance template
                    sh """
                        gcloud compute instance-groups managed rolling-action start-update ${MIG_JAKARTA} \
                            --image asia-southeast2-docker.pkg.dev/${GCP_PROJECT_ID}/${REPOSITORY_NAME}/${IMAGE_NAME}:latest \
                            --image-project ${GCP_PROJECT_ID} \
                            --region ${REGION_JAKARTA} \
                            --mode=single
                    """
                }
            }
        }

        stage('Cleanup Docker Images') {
            steps {
                script {
                    sh """
                        docker image prune -f
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment to MIGs completed successfully!!'
        }
        failure {
            echo 'Deployment failed. Check the logs for details.'
        }
    }
}
