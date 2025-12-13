pipeline {
    agent {
        node {
            label 'kr-jenkins-slave-1'
        }

    }

    

    environment {
        TELEGRAM_TOKEN = credentials('TELEGRAM_TOKEN')
        TELEGRAM_CHAT_ID = credentials('TELEGRAM_CHAT_ID')
        GITHUB_REPO = "https://github.com/natnael-ta/cbe-superapp-deployment.git"
        GITLAB_REPO = "https://gitlab.cbe.com.et/NatnaelTadesse/cbe-superapp-deployment.git"
        PROJECT_ROOT_PATH = "/data/cbe-super-app/"
    }

    stages {
       stage('Cloning Git') {
            steps {
                checkout([
                    $class: 'GitSCM', 
                    branches: [[name: "*/master"]], 
                    doGenerateSubmoduleConfigurations: false, 
                    extensions: [], 
                    submoduleCfg: [], 
                    userRemoteConfigs: [[
                        credentialsId:  'GITHUB_CRED', 
                        url: env.GITHUB_REPO
                    ]]
                ])  
            }
        }
stage('Sync & Push to GitLab') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'GITHUB_CRED',
                usernameVariable: 'GH_USER',
                passwordVariable: 'GH_PASS'
            ),
            usernamePassword(
                credentialsId: 'GITLAB_CRED',
                usernameVariable: 'GL_USER',
                passwordVariable: 'GL_PASS'
            )
        ]) {
            sh '''
                set -e

                echo "Reconfiguring GitHub remote with credentials..."
                git remote set-url origin https://${GH_USER}:${GH_PASS}@github.com/natnael-ta/cbe-superapp-deployment.git

                echo "Fetching latest changes from GitHub..."
                git fetch origin --prune

                echo "Resetting local master to origin/master..."
                git checkout master
                git reset --hard origin/master

                echo "Fetching tags from GitHub..."
                git fetch origin --tags --force

                echo "Configuring GitLab remote..."
                git remote remove gitlab || true
                git remote add gitlab https://${GL_USER}:${GL_PASS}@gitlab.cbe.com.et/NatnaelTadesse/cbe-superapp-deployment.git

                echo "Pushing branches to GitLab..."
                git push gitlab --force --all

                echo "Pushing tags to GitLab..."
                git push gitlab --force --tags
            '''
        }
    }
}



    post {
        success {
            sh """
            curl -X POST \
            https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage \
            -d chat_id=${TELEGRAM_CHAT_ID}  \
            -d text="Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            """
        }
        failure {
            sh """
            curl -X POST \
            https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage \
            -d chat_id=${TELEGRAM_CHAT_ID}  \
            -d text="Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            """
        }
    }
}

