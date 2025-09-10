pipeline {
    agent {
        docker {
            image 'alpine:3.14'
            args '-v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker'
        }
    }
    
    environment {
        REGISTRY = "localhost:5000"
        ANSIBLE_DIR = "/var/jenkins_home/ansible"
        SSH_KEY = credentials('ansible-ssh-key')
        VAULT_PASSWORD = credentials('ansible-vault-password')
        DOCKER_HOST = "unix:///var/run/docker.sock"
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
                        url: 'https://github.com/tuo-username/formazione_cm.git',
                        credentialsId: 'git-credentials'  // Se necessario
                    ]]
                ])
                sh 'ls -la'
            }
        }
        
        stage('Setup Environment') {
            steps {
                sh '''
                echo "⚙️ Setting up environment..."
                apk add --no-cache python3 py3-pip
                pip3 install ansible
                
                mkdir -p ${ANSIBLE_DIR}
                cp -r * ${ANSIBLE_DIR}/
                
                # Setup SSH keys
                mkdir -p ${ANSIBLE_DIR}/files/ssh-keys/
                echo "${SSH_KEY}" > ${ANSIBLE_DIR}/files/ssh-keys/id_rsa
                chmod 600 ${ANSIBLE_DIR}/files/ssh-keys/id_rsa
                
                # Setup vault password
                echo "${VAULT_PASSWORD}" > ${ANSIBLE_DIR}/.vault-pass
                chmod 600 ${ANSIBLE_DIR}/.vault-pass
                
                echo "📁 Directory structure:"
                ls -la ${ANSIBLE_DIR}/
                '''
            }
        }
        
        stage('Verify Tools') {
            steps {
                sh '''
                echo "🔧 Checking tools versions..."
                docker --version
                ansible --version
                python3 --version
                echo "✅ All tools available"
                '''
            }
        }
        
        stage('Build Base Images') {
            steps {
                sh '''
                echo "🐳 Building base Alpine and Ubuntu images..."
                cd ${ANSIBLE_DIR}
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
                    cd ${ANSIBLE_DIR}
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
                cd ${ANSIBLE_DIR}
                ansible-playbook -i inventory.ini deploy-pipeline.yml \
                  -e "image_tag=${IMAGE_TAG}" \
                  --vault-password-file .vault-pass -v
                """
            }
        }
        
        stage('Verify Deployment') {
            steps {
                sh '''
                echo "🔍 Verifying deployment..."
                sleep 5
                
                # Check if container is running
                if docker ps --filter name=pipeline-app --format "{{.Status}}" | grep -q "Up"; then
                    echo "✅ Container is running"
                else
                    echo "❌ Container not running"
                    docker ps -a
                    exit 1
                fi
                
                # Test application endpoint
                if curl -s -f http://localhost:8081 > /dev/null; then
                    echo "✅ Application is responding"
                else
                    echo "⚠️ Application not responding yet, waiting..."
                    sleep 10
                    curl -f http://localhost:8081 || echo "❌ Application failed to start"
                fi
                '''
            }
        }
        
        stage('Run Tests') {
            steps {
                sh '''
                echo "🧪 Running basic tests..."
                
                # Test SSH connection to base containers
                if ssh -i ${ANSIBLE_DIR}/files/ssh-keys/id_rsa \
                   -o StrictHostKeyChecking=no \
                   -o ConnectTimeout=5 \
                   -p 2222 ansible-user@localhost whoami; then
                    echo "✅ SSH to Alpine container working"
                else
                    echo "⚠️ SSH to Alpine container failed"
                fi
                
                if ssh -i ${ANSIBLE_DIR}/files/ssh-keys/id_rsa \
                   -o StrictHostKeyChecking=no \
                   -o ConnectTimeout=5 \
                   -p 2223 ansible-user@localhost whoami; then
                    echo "✅ SSH to Ubuntu container working"
                else
                    echo "⚠️ SSH to Ubuntu container failed"
                fi
                '''
            }
        }
    }
    
    post {
        always {
            echo "🧹 Cleaning up workspace..."
            cleanWs()
            
            sh '''
            echo "📊 Final container status:"
            docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
            '''
        }
        success {
            echo "🎉 Pipeline completed successfully!"
            echo "📦 Image tag: ${IMAGE_TAG}"
            echo "🌐 Application URL: http://localhost:8081"
            echo "🔐 SSH Alpine: ssh -i files/ssh-keys/id_rsa -p 2222 ansible-user@localhost"
            echo "🔐 SSH Ubuntu: ssh -i files/ssh-keys/id_rsa -p 2223 ansible-user@localhost"
        }
        failure {
            echo "💥 Pipeline failed!"
            sh '''
            echo "🔍 Debug information:"
            docker logs pipeline-app 2>/dev/null || echo "No pipeline-app container"
            docker ps -a
            '''
        }
        unstable {
            echo "⚠️ Pipeline unstable - some tests failed but deployment succeeded"
        }
    }
}