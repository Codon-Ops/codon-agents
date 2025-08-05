pipeline {
    agent any

    environment {
        EC2_HOST   = '100.28.76.68'
        EC2_USER   = 'ubuntu'
        REPO_DIR   = '/home/ubuntu/codon-agents'
        CRED_ID    = 'codon-agents-ui'
    }

    stages {
        stage('Stop Services') {
            steps {
                echo '🛑 Stopping Docker containers on EC2...'
                withCredentials([sshUserPrivateKey(credentialsId: env.CRED_ID, keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -i $SSH_KEY ${EC2_USER}@${EC2_HOST} "
                            # ensure Git won’t refuse to run here
                            git config --global --add safe.directory ${REPO_DIR} && \
                            cd ${REPO_DIR} && \
                            sudo docker-compose down || echo 'No containers to stop'
                        "
                    '''
                }
            }
        }

        stage('Update Code') {
            steps {
                echo '📥 Pulling latest code on EC2...'
                withCredentials([sshUserPrivateKey(credentialsId: env.CRED_ID, keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -i $SSH_KEY ${EC2_USER}@${EC2_HOST} "
                            git config --global --add safe.directory ${REPO_DIR} && \
                            cd ${REPO_DIR} && \
                            git pull origin main
                        "
                    '''
                }
            }
        }

        stage('Start Services') {
            steps {
                echo '🚀 Starting Docker containers on EC2...'
                withCredentials([sshUserPrivateKey(credentialsId: env.CRED_ID, keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -i $SSH_KEY ${EC2_USER}@${EC2_HOST} "
                            cd ${REPO_DIR} && \
                            sudo docker-compose up -d
                        "
                    '''
                }
            }
        }

        stage('Health Check') {
            steps {
                echo '🏥 Checking container status on EC2...'
                withCredentials([sshUserPrivateKey(credentialsId: env.CRED_ID, keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -i $SSH_KEY ${EC2_USER}@${EC2_HOST} "
                            cd ${REPO_DIR} && \
                            sleep 15 && \
                            sudo docker-compose ps
                        "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Remote deployment completed successfully!'
        }
        failure {
            echo '❌ Remote deployment failed!'
            withCredentials([sshUserPrivateKey(credentialsId: env.CRED_ID, keyFileVariable: 'SSH_KEY')]) {
                sh '''
                    ssh -o StrictHostKeyChecking=no -i $SSH_KEY ${EC2_USER}@${EC2_HOST} "
                        cd ${REPO_DIR} && \
                        sudo docker-compose logs --tail=20
                    "
                '''
            }
        }
        always {
            echo '📊 Final container status on EC2:'
            withCredentials([sshUserPrivateKey(credentialsId: env.CRED_ID, keyFileVariable: 'SSH_KEY')]) {
                sh '''
                    ssh -o StrictHostKeyChecking=no -i $SSH_KEY ${EC2_USER}@${EC2_HOST} "
                        cd ${REPO_DIR} && \
                        sudo docker-compose ps || echo 'Could not get container status'
                    "
                '''
            }
        }
    }
}
