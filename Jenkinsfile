pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy to EC2') {
            steps {
                sshagent (credentials: ['codon-agent-ui-1']) {
                    sh '''
                        # Shut down containers
                        ssh -o StrictHostKeyChecking=no ubuntu@3.93.224.218 "cd /home/ubuntu/langgraph-agent-demo && /usr/bin/docker-compose down -v"

                        # Sync updated repo
                        rsync -vrzhe "ssh -o StrictHostKeyChecking=No" --rsync-path="/usr/bin/sudo /usr/bin/rsync" --partial --protocol=30 --timeout=300 --exclude-from='exclude-file.txt' ./ ubuntu@3.93.224.218:/home/ubuntu/langgraph-agent-demo/

                        # Start containers again
                        ssh -o StrictHostKeyChecking=no ubuntu@3.93.224.218 "cd /home/ubuntu/langgraph-agent-demo && /usr/bin/docker-compose up -d"
                    '''
                }
            }
        }
    }
}
