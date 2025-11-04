pipeline {
    agent any

    environment {
        SSH_CRED = 'Server-Key'               // Jenkins SSH key for server login
        SSH_USER = 'ubuntu'                        // SSH username
        SSH_HOST = '13.203.74.135'                // Remote server IP
        DEPLOY_PATH = '/var/www/html'              // Deployment directory
        BRANCH = 'main'                            // Branch to deploy
        REPO_URL = 'https://github.com/ishs-25/react-portfolio' // Public repo URL
    }

    triggers {
        githubPush()  // Trigger on GitHub push
    }

    stages {

        stage('Checkout Jenkinsfile') {
            steps {
                checkout scm
                script {
                    env.REPO_NAME = sh(script: "basename -s .git ${REPO_URL}", returnStdout: true).trim()
                    echo "📂 Repo name detected: ${env.REPO_NAME}"
                }
            }
        }

        stage('Prepare Remote Environment') {
            steps {
                echo '⚙️ Preparing remote server...'
                sshagent (credentials: [env.SSH_CRED]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${SSH_USER}@${SSH_HOST} << 'EOF'
set -e

echo "Updating packages..."
sudo apt update -y

# Install Git
command -v git > /dev/null || sudo apt install -y git

# Install Node.js (LTS)
command -v node > /dev/null || (curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash - && sudo apt install -y nodejs)

# Install Nginx
command -v nginx > /dev/null || (sudo apt install -y nginx && sudo systemctl enable nginx && sudo systemctl start nginx)

echo "✅ Remote server ready!"
EOF
                    """
                }
            }
        }

        stage('Clone & Build Application') {
            steps {
                echo '🔄 Cloning and building on remote server...'
                sshagent (credentials: [env.SSH_CRED]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${SSH_USER}@${SSH_HOST} << 'EOF'
set -e
cd ~

# Remove old repo if exists
[ -d "${REPO_NAME}" ] && sudo rm -rf ${REPO_NAME}

echo "Cloning public repository..."
git clone -b ${BRANCH} ${REPO_URL} ${REPO_NAME}

cd ${REPO_NAME}

echo "Installing npm dependencies..."
npm install

echo "Building project..."
npm run build
echo "✅ Build completed!"
EOF
                    """
                }
            }
        }

        stage('Deploy Build') {
            steps {
                echo '🚀 Deploying build...'
                sshagent (credentials: [env.SSH_CRED]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${SSH_USER}@${SSH_HOST} << EOF
set -e

sudo mkdir -p ${DEPLOY_PATH}
sudo rm -rf ${DEPLOY_PATH}/*
sudo cp -r ${REPO_NAME}/build/* ${DEPLOY_PATH}/

echo "✅ Deployment successful!"
EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo '🎉 Deployment completed successfully!'
        }
        failure {
            echo '❌ Deployment failed. Check Jenkins logs.'
        }
    }
}