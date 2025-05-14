pipeline {
    agent any

    environment {
        NODE_ENV = 'qa'
        EC2_USER = 'ubuntu'
        EC2_IP = '3.211.128.84'
        REMOTE_PATH = '/home/ubuntu/jenkins'
        SSH_KEY = credentials('ssh-key-ec2')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'qa', url: 'https://github.com/diegobejar1011/jenkins.git'
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
                sh """
                ssh -i $SSH_KEY -o StrictHostKeyChecking=no $EC2_USER@$EC2_IP '
                    cd $REMOTE_PATH &&
                    git pull origin qa &&
                    npm ci &&
                    pm2 restart health-api || pm2 start server.js --name health-api
                '
                """
            }
        }
    }
}
