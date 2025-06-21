pipeline{
    agent any

    tools{
        nodejs 'NodeJS'
    }

    environment{
        VM_HOST = '192.168.1.3'
        VM_USER = 'mikeross'
        DEPLOY_PATH = '/var/www'
    }
    stages{
        stage('Checkout'){
            steps{
                echo 'Checking out code from github...'
            }
        }
        stage('Install Dependencies'){
            steps{
                echo 'Installing dependencies...'
                powershell  'npm install'
            }
        }
        stage('Build'){
            steps{
                echo 'Building react application...'
                powershell  'npm run build'
            }
        }
        stage('Test'){
            steps{
                echo 'Running tests...'
            }
        }
        stage('Deploy to Ubuntu VM'){
            steps{
                echo 'Deploying the application...'

                // for linux based agents, you can use the sshagent plugin to handle SSH keys
                // sshagent(['vm-ubuntu-mikeross-unpw']) {
                //     // Copying build files to the VM
                //     bat """
                //         scp -o StrictHostKeyChecking=no -r build/* ${VM_USER}@${VM_HOST}:${DEPLOY_PATH}

                //     """
                //     // Restarting the nginx on the VM
                //     bat """
                //         ssh -o StrictHostKeyChecking=no ${VM_USER}@${VM_HOST} 'sudo systemctl reload nginx'
                //     """
                // }

                // For Windows agents, using withCredentials to handle SSH keys because the agent is running on a Windows machine and  StringIndexOutOfBoundsException(environment variable parsing exception) can occur while using sshagent on Windows
                withCredentials([sshUserPrivateKey(credentialsId: 'vm-ubuntu-mikeross-unpw', keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER')]) {
                   bat '''
                        echo 'Copying build files to the VM...'
                        ssh -i "%SSH_KEY%" -o StrictHostKeyChecking=no %SSH_USER%@192.168.1.3 "mkdir -p /var/www/build"
                        scp -i "%SSH_KEY%" -o StrictHostKeyChecking=no -r build/* %SSH_USER%@192.168.1.3:/var/www/build/
                        echo "Restarting nginx..."
                        ssh -i "%SSH_KEY%" -o StrictHostKeyChecking=no %SSH_USER%@192.168.1.3 "sudo systemctl restart nginx"
                        echo "Checking nginx status..."
                        ssh -i "%SSH_KEY%" -o StrictHostKeyChecking=no %SSH_USER%@192.168.1.3 "sudo systemctl status nginx --no-pager"
                    '''
                }
            }
        }
    }   
    // Post actions can be added here if needed
    post {
        always {
            echo 'Pipeline execution completed.'
            // Archive build artifacts or perform cleanup if necessary
            archiveArtifacts artifacts: 'build/**', allowEmptyArchive: true
        }
        success {
            echo 'Deployment successful!'
            echo 'Application is live at http://${VM_HOST}'
        }
        failure {
            echo 'Deployment failed!'
            echo 'Please check the logs for errors.'
        }
        cleanup {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}