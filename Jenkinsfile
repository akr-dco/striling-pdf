pipeline {
    agent any

    environment {
        TARGET_USER = "cicd"
        PROD_HOST   = "192.168.150.150"
        STAGING_HOST= "192.168.192.78"
    }

    stages {

        stage('Resolve Target') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.TARGET_HOST = env.PROD_HOST
                        env.TARGET_DIR  = "/home/cicd/pdf"
                        env.DEPLOY_ENV  = "PRODUCTION"
                    } 
                    else if (env.BRANCH_NAME == 'staging') {
                        env.TARGET_HOST = env.STAGING_HOST
                        env.TARGET_DIR  = "/home/cicd/pdf"
                        env.DEPLOY_ENV  = "STAGING"
                    } 
                    else {
                        error "Branch ${env.BRANCH_NAME} is not allowed to deploy"
                    }
                }

                echo "ENV    : ${DEPLOY_ENV}"
                echo "HOST   : ${TARGET_HOST}"
                echo "DIR    : ${TARGET_DIR}"
            }
        }

        stage('Prepare Target') {
            steps {
                sshagent(['privatekey-akr']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${TARGET_USER}@${TARGET_HOST} '
                        mkdir -p ${TARGET_DIR}
                    '
                    """
                }
            }
        }

        stage('Sync Repository') {
            steps {
                sshagent(['privatekey-akr']) {
                    sh """
                    rsync -avz --delete \
                        --exclude '.git' \
                        --exclude '.jenkins' \
                        ./ \
                        ${TARGET_USER}@${TARGET_HOST}:${TARGET_DIR}/
                    """
                }
            }
        }

        stage('Deploy Docker Compose') {
            steps {
                sshagent(['privatekey-akr']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${TARGET_USER}@${TARGET_HOST} '
                        cd ${TARGET_DIR} &&
                        docker compose down &&
                        docker compose up -d
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ ${DEPLOY_ENV} DEPLOY SUCCESS (${env.BRANCH_NAME})"
        }
        failure {
            echo "❌ ${DEPLOY_ENV} DEPLOY FAILED (${env.BRANCH_NAME})"
        }
    }
}
