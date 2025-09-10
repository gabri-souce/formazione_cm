pipeline {
    agent any
    
    environment {
        REGISTRY = "localhost:5000"
        ANSIBLE_DIR = "${WORKSPACE}"
        SSH_KEY_FILE = "${WORKSPACE}/files/ssh-keys/id_rsa"
        VAULT_PASSWORD_FILE = "${WORKSPACE}/.vault-pass"
    }
    
    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    extensions: [[$class: 'CleanCheckout']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/gabri-souce/formazione_cm.git'
                    ]]
                ])
                sh 'ls -la'
            }
        }
        
        stage('Setup Environment') {
            steps {
                script {
                    // Setup SSH key from credentials
                    withCredentials([sshUserPrivateKey(
                        credentialsId: 'ansible-ssh-key',
                        keyFileVariable: 'SSH_KEY_CONTENT'
                    )]) {
                        writeFile file: env.SSH_KEY_FILE, text: env.SSH_KEY_CONTENT
                        sh "chmod 600 ${SSH_KEY_FILE}"
                    }
                    
                    // Setup vault password from credentials
                    withCredentials([string(
                        credentialsId: 'ansible-vault-password',
                        variable: 'VAULT_PASSWORD_CONTENT'
                    )]) {
                        writeFile file: env.VAULT_PASSWORD_FILE, text: env.VAULT_PASSWORD_CONTENT
                        sh "chmod 600 ${VAULT_PASSWORD_FILE}"
                    }
                    
                    sh """
                    echo "📁 Directory structure:"
                    ls -la
                    ls -la files/ssh-keys/
                    """
                }
            }
        }
        
        stage('Verify Tools') {
            steps {
                sh '''
                echo "🔧 Checking tools versions..."
                which docker || echo "Docker not in PATH"
                which ansible || echo "Ansible not in PATH"
                which git || echo "Git not in PATH"
                python3 --version || echo "Python3 not found"
                '''
            }
        }
        
        stage('Build Base Images') {
            steps {
                sh '''
                echo "🐳 Building base Alpine and Ubuntu images..."
                # Use host docker instead of trying to use docker inside container
                docker --version
                ansible-playbook -i inventory.ini container-playbook.yml \
                  --vault-password-file .vault-pass \
                  --tags build
                echo "✅ Base images built successfully"
                '''
            }
        }
        
        stage('Build Application Image') {
            steps {
                script {
                    def version = sh(script: 'date +%Y%m%d%H%M%S', returnStdout: true).trim()
                    env.IMAGE_TAG = version
                    
                    echo "🏗️ Building application image with tag: ${version}"
                    
                    sh """
                    ansible-playbook -i inventory.ini build-pipeline.yml \
                      -e "image_tag=${version}" \
                      --vault-password-file .vault-pass -v
                    """
                }
            }
        }
        
        stage('Deploy Application') {
            steps {
                sh """
                echo "🚀 Deploying application version: ${IMAGE_TAG}"
                ansible-playbook -i inventory.ini deploy-pipeline.yml \
                  -e "image_tag=${IMAGE_TAG}" \
                  --vault-password-file .vault-pass -v
                """
            }
        }
    }
    
    post {
        always {
            echo "🧹 Cleaning up..."
            // Cleanup sensitive files
            sh """
            rm -f ${SSH_KEY_FILE} ${VAULT_PASSWORD_FILE} 2>/dev/null || true
            """
        }
        success {
            echo "🎉 Pipeline completed successfully!"
            echo "📦 Image tag: ${IMAGE_TAG}"
            echo "🌐 Application URL: http://localhost:8081"
        }
        failure {
            echo "💥 Pipeline failed!"
            sh '''
            echo "🔍 Debug information:"
            docker ps -a || true
            '''
        }
    }
}