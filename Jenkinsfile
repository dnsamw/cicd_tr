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
                   powershell '''
                        Write-Host "Fixing SSH key permissions..."
                        $keyPath = $env:SSH_KEY
                        
                        # Remove inheritance and set proper permissions
                        $acl = Get-Acl $keyPath
                        $acl.SetAccessRuleProtection($true, $false)
                        $accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule($env:USERNAME, "Read", "Allow")
                        $acl.SetAccessRule($accessRule)
                        Set-Acl $keyPath $acl
                        
                        Write-Host "Copying build files to the VM..."
                        & ssh -i "$env:SSH_KEY" -o StrictHostKeyChecking=no "$env:SSH_USER@192.168.1.3" "mkdir -p /var/www/build"
                        & scp -i "$env:SSH_KEY" -o StrictHostKeyChecking=no -r build/* "$env:SSH_USER@192.168.1.3:/var/www/build/"
                        
                        Write-Host "Restarting nginx..."
                        & ssh -i "$env:SSH_KEY" -o StrictHostKeyChecking=no "$env:SSH_USER@192.168.1.3" "sudo systemctl restart nginx"
                        
                        Write-Host "Checking nginx status..."
                        & ssh -i "$env:SSH_KEY" -o StrictHostKeyChecking=no "$env:SSH_USER@192.168.1.3" "sudo systemctl status nginx --no-pager"
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