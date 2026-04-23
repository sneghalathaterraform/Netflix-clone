environment {
    S3_BUCKET      = 'netflix-clone-devops-snegha'    // ← CHANGE THIS
    APP_NAME       = 'netflix-clone'           // leave this as is
    DEPLOY_DIR     = '/opt/tomcat/webapps/ROOT'// leave this as is
    ANSIBLE_HOST   = '18.205.28.50'      // ← CHANGE THIS
}

stage('Clone Repository') {
    steps {
        echo '>>> Cloning repository...'
        git branch: 'main', url: 'https://github.com/sneghalathaterraform/Netflix-clone.git'
    }
}