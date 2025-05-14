pipeline {
    agent any

    environment {
        SSH_KEY = credentials('ssh-key-ec2')
    }

    stages {
        stage('Set Environment') {
            steps {
                script {
                    def branch = env.GIT_BRANCH;

                    switch(branch) {
                        case 'main':
                            env.NODE_ENV = 'production'
                            env.EC2_USER = 'ubuntu'
                            env.EC2_IP = '3.211.128.84'
                            env.REMOTE_PATH = '/home/ubuntu/jenkins'
                            break
                        case 'dev':
                            env.NODE_ENV = 'dev'
                            env.EC2_USER = 'ubuntu'
                            env.EC2_IP = '34.196.198.8'
                            env.REMOTE_PATH = '/home/ubuntu/jenkins'
                            break
                        case 'qa':
                            env.NODE_ENV = 'qa'
                            env.EC2_USER = 'ubuntu'
                            env.EC2_IP = '52.1.107.36'
                            env.REMOTE_PATH = '/home/ubuntu/jenkins'
                            break
                        default:
                            error "Unsupported branch: ${branch}"
                    }
                }
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
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
                    git pull origin ${env.GIT_BRANCH} &&
                    npm ci &&
                    pm2 restart health-api || pm2 start server.js --name health-api
                '
                """
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed!'
        }
        success {
            echo "Deployed to ${env.NODE_ENV} environment on ${env.EC2_IP}"
        }
    }
}