pipeline {
    agent any

    environment {
        APP_REPO       = 'https://github.com/jeevakalaiselvam/rga-app.git'
        TERRAFORM_REPO = 'https://github.com/jeevakalaiselvam/rga-terraform.git'
        DEPLOYED_AT    = sh(script: 'date "+%Y-%m-%d %H:%M:%S"', returnStdout: true).trim()
    }

    stages {

        stage('Checkout App') {
            steps {
                echo '==> Pulling application code...'
                dir('rga-app') {
                    git url: "${APP_REPO}", branch: 'main'
                }
            }
        }

        stage('Run Tests') {
            steps {
                echo '==> Running unit tests...'
                dir('rga-app') {
                    sh '''
                        npm install
                        npm test
                    '''
                }
            }
        }

        stage('Checkout Terraform') {
            steps {
                echo '==> Pulling Terraform code...'
                dir('rga-terraform') {
                    git url: "${TERRAFORM_REPO}", branch: 'main'
                }
            }
        }

        stage('Terraform Init') {
            steps {
                echo '==> Initializing Terraform...'
                dir('rga-terraform') {
                    sh 'terraform init'
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                echo '==> Running Terraform plan...'
                dir('rga-terraform') {
                    sh """
                        terraform plan \
                          -var="app_version=1.0.0" \
                          -var="environment=dev" \
                          -var="deployed_at=${DEPLOYED_AT}"
                    """
                }
            }
        }

        stage('Approval') {
            steps {
                echo '==> Waiting for approval...'
                input message: 'Terraform plan complete. Apply to dev?',
                      ok: 'Yes, Apply'
            }
        }

        stage('Terraform Apply') {
            steps {
                echo '==> Applying Terraform...'
                dir('rga-terraform') {
                    sh """
                        terraform apply -auto-approve \
                          -var="app_version=1.0.0" \
                          -var="environment=dev" \
                          -var="deployed_at=${DEPLOYED_AT}"
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo '==> Verifying deployment record...'
                sh 'cat /tmp/rga-deployment.txt'
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed — check logs above.'
        }
        always {
            echo '==> Cleaning up workspace...'
            cleanWs()
        }
    }
}