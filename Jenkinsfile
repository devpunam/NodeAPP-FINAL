pipeline {
    agent any

    environment {
        EC2_HOST = "3.109.210.146"
        SSH_CREDENTIAL_ID = "ec2-key"
        REMOTE_USER = "ubuntu"
        REMOTE_PATH = "/home/ubuntu/app"
        WEB_ROOT = "/var/www/html"
        SERVER = "nginx"
    }

    stages {
        stage("Build") {
            steps {
                echo "Installing dependencies"
                sh "npm install"
                echo "Building the app"
                sh "npm run build"
                echo "Building successful"
            }
        }

        stage("Deploy") {
            steps {
                echo "Starting deployment"
                sshagent(credentials: [SSH_CREDENTIAL_ID]) {
                    sh """
                        echo "Creating Remote Directory"
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${EC2_HOST} 'mkdir -p ${REMOTE_PATH}'

                        echo "Copying Distribution to Remote path"
                        scp -o StrictHostKeyChecking=no -r dist/* ${REMOTE_USER}@${EC2_HOST}:${REMOTE_PATH}/

                        echo "Restarting Nginx"
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${EC2_HOST} "
                            sudo rm -rf ${WEB_ROOT}/*
                            sudo cp -r ${REMOTE_PATH}/* ${WEB_ROOT}/
                            sudo systemctl restart ${SERVER}
                        "
                    """
                }
            }
        }
    }
}
