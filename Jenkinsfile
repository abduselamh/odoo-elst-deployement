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
        stage('Push to GitLab') {
            steps {
                // Push all refs to GitLab using credentials
                withCredentials([usernamePassword(credentialsId: 'GITLAB_CRED', usernameVariable: 'GL_USER', passwordVariable: 'GL_PASS')]) {
                    // Push all branches and tags to GitLab (avoid hidden refs)
{
            sh """
            set -e

            echo "Fetching latest changes from GitHub..."
            git fetch origin --prune

            echo "Resetting local branch to match GitHub master..."
            git checkout master
            git reset --hard origin/master

            echo "Fetching all tags from GitHub..."
            git fetch origin --tags --force

            echo "Configuring GitLab remote..."
            git remote remove gitlab || true
            git remote add gitlab https://${GL_USER}:${GL_PASS}@${GITLAB_REPO.replace('https://', '')}

            echo "Pushing all branches to GitLab..."
            git push gitlab --force --all

            echo "Pushing all tags to GitLab..."
            git push gitlab --force --tags
            """
        }
                }
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

