pipeline {
    agent any

    environment {
        GCP_PROJECT_ID = 'belajar-terraform-dan-ansible'
        IMAGE_NAME = 'nodejs-app'
        REGION_SINGAPORE = 'asia-southeast1'
        ZONE_SINGAPORE = 'asia-southeast1-a'
        REGION_JAKARTA = 'asia-southeast2'
        ZONE_JAKARTA = 'asia-southeast2-a'
        REPOSITORY_NAME = 'nodejs-docker-image-repo'
        SG_VM_INSTANCE = 'sg-nodeapp-instance'
        JKT_VM_INSTANCE = 'jkt-nodeapp-instance'
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
        stage('Deploy to Singapore VM') {
            steps {
                script {
                    // SSH ke VM instance di Singapore dan pull Docker image
                    sh """
                        export PATH=/opt/google-cloud-sdk/bin:\$PATH                    
                        gcloud compute ssh ${SG_VM_INSTANCE} --zone ${ZONE_SINGAPORE} --command 'docker pull ${REGION_JAKARTA}-docker.pkg.dev/${GCP_PROJECT_ID}/${REPOSITORY_NAME}/${IMAGE_NAME}:latest && \
                            docker run -d --name nodejs-app ${REGION_JAKARTA}-docker.pkg.dev/${GCP_PROJECT_ID}/${REPOSITORY_NAME}/${IMAGE_NAME}:latest'
                    """
                }
            }
        }

        stage('Deploy to Jakarta VM') {
            steps {
                script {
                    // SSH ke VM instance di Jakarta dan pull Docker image
                    sh """
                        export PATH=/opt/google-cloud-sdk/bin:\$PATH                    
                        gcloud compute ssh ${JKT_VM_INSTANCE} --zone ${ZONE_JAKARTA} --command 'docker pull ${REGION_JAKARTA}-docker.pkg.dev/${GCP_PROJECT_ID}/${REPOSITORY_NAME}/${IMAGE_NAME}:latest && \
                            docker run -d --name nodejs-app ${REGION_JAKARTA}-docker.pkg.dev/${GCP_PROJECT_ID}/${REPOSITORY_NAME}/${IMAGE_NAME}:latest'
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
