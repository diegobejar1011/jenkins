pipeline {
    agent any

    environment {
        SSH_KEY = credentials('ssh-key-ec2')
    }

    stages {
        stage('Set Environment') {
            steps {
                script {
                    // Elimina el prefijo "origin/" si está presente
                    def branch = env.GIT_BRANCH.replaceFirst(/^origin\//, '')

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

                    // Guarda el nombre limpio de la rama para usar en el git pull
                    env.BRANCH_NAME_CLEAN = branch
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
                        git checkout ${env.BRANCH_NAME_CLEAN} &&
                        git pull --rebase origin ${env.BRANCH_NAME_CLEAN} &&
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
