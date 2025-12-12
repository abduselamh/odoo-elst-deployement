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
        GITHUB_CRED=credentials("GITHUB_CRED")
        GITLAB_CRED=credentials("GITLAB_CRED")
    }

    stages {
        stage('Clone GitHub Repo') {
            steps {
                // Clone the GitHub repository using a Personal Access Token (PAT)
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/master']], // using master as default branch
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [],
                    userRemoteConfigs: [[
                        url: env.GITHUB_REPO,
                        credentialsId: env.GITHUB_TOKEN
                    ]]
                ])
            }
        }
        stage('Push to GitLab') {
            steps {
                // Push all refs to GitLab using credentials
                withCredentials([usernamePassword(credentialsId: env.GITLAB_CRED, usernameVariable: 'GL_USER', passwordVariable: 'GL_PASS')]) {
                    sh "git remote add gitlab https://${GL_USER}:${GL_PASS}@${GITLAB_REPO.replace('https://', '')}"
                    sh 'git push gitlab --mirror'
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

