pipeline {
    agent any

    environment {
        S3_BUCKET    = 'netflix-clone-devops-snegha'
        APP_NAME     = 'netflix-clone'
        DEPLOY_DIR   = '/opt/tomcat/webapps/ROOT'
        ANSIBLE_HOST = '18.205.28.50'
    }

    stages {

        stage('Clone Repository') {
            steps {
                echo '>>> Cloning repository...'
                git branch: 'main',
                    url: 'https://github.com/sneghalathaterraform/Netflix-clone.git'
            }
        }

        stage('Validate App') {
            steps {
                echo '>>> Validating HTML file...'
                sh 'ls -la'
                sh 'cat index.html | grep -c "<html" && echo "HTML valid"'
            }
        }

        stage('Package App') {
            steps {
                echo '>>> Packaging app as ZIP...'
                sh 'zip -r ${APP_NAME}.zip . -x "*.git*" "Jenkinsfile" "*.md"'
                sh 'ls -lh ${APP_NAME}.zip'
            }
        }

       stage('Upload to S3') {
            steps {
                echo '>>> Uploading to S3...'
                withAWS(credentials: 'aws-credentials', region: 'us-east-1') {
                    sh """
                        aws s3 cp ${APP_NAME}.zip s3://${S3_BUCKET}/releases/${APP_NAME}-${BUILD_NUMBER}.zip
                    """
                }
            }
        }

        stage('Deploy via Ansible') {
            steps {
                echo '>>> Deploying with Ansible...'
                sh """
                    ansible-playbook -i /etc/ansible/inventory/hosts \
                        /etc/ansible/playbooks/deploy.yml \
                        --extra-vars "build_number=${BUILD_NUMBER} s3_bucket=${S3_BUCKET} app_name=${APP_NAME}"
                """
            }
        }

        stage('Health Check') {
            steps {
                echo '>>> Running health check...'
                sh """
                    sleep 10
                    HTTP_CODE=\$(curl -s -o /dev/null -w "%{http_code}" http://${ANSIBLE_HOST}:8080/)
                    if [ "\$HTTP_CODE" -eq 200 ]; then
                        echo "Health check PASSED: HTTP \$HTTP_CODE"
                    else
                        echo "Health check FAILED: HTTP \$HTTP_CODE"
                        exit 1
                    fi
                """
            }
        }

    }

    post {
        success {
            echo 'DEPLOYMENT SUCCESSFUL — Netflix Clone is LIVE at http://18.205.28.50:8080'
        }
        failure {
            echo 'DEPLOYMENT FAILED — Triggering auto-heal...'
            sh 'ansible-playbook -i /etc/ansible/inventory/hosts /etc/ansible/playbooks/heal.yml || true'
        }
    }
}
