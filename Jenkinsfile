pipeline {
    agent any

    environment {
        NODE_ENV = 'production'
        EC2_USER = 'ubuntu'
        EC2_IP = '98.81.79.234'
        REMOTE_PATH = '/home/ubuntu/jenkins'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/diegobejar1011/jenkins.git'
            }
        }

        stage('Build') {
            steps {
                sh 'rm -rf node_modules'
                sh 'npm ci'
            }
        }

        stage('Deploy') {
            steps {
                sshagent(credentials: ['ssh-key-ec2']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} '
                        cd ${REMOTE_PATH} &&
                        git pull origin main &&
                        npm ci &&
                        if pm2 list | grep -q "health-api"; then
                            pm2 restart health-api
                        else
                            pm2 start server.js --name health-api
                        fi
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Despliegue exitoso.'
        }
        failure {
            echo '❌ Hubo un error en el pipeline.'
        }
    }
}
