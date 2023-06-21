pipeline {
    agent any

    environment {
        NAME = "spring-app"
        VERSION = "${env.BUILD_ID}-${env.GIT_COMMIT}"
        IMAGE_REPO = "praveensirvi"
        ARGOCD_TOKEN = credentials('argocd-token')
        GITEA_TOKEN = credentials('gitea-token')
    }

    tools { 
        maven 'maven-3.8.6' 
    }
    stages {
        stage('Checkout git') {
            steps {
              git 'https://github.com/praveensirvi1212/DevOps_MasterPiece-CI-with-Jenkins.git'
            }
        }
        
        stage ('Build & JUnit Test') {
            steps {
                sh 'mvn clean package' 
            }
        }

        stage('SonarQube Analysis'){
            steps{
                withSonarQubeEnv('SonarQube-server') {
                        sh 'mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=devsecops-project-key \
                        -Dsonar.host.url=$sonarurl \
                        -Dsonar.login=$sonarlogin'
                }
            }
        }

        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }
        
        stage('Artifact Upload') {
            steps {
      	        
            }
        }

        stage('Docker  Build') {
            steps {
      	        sh 'docker build -t ${IMAGE_REPO}/${NAME}:${VERSION} .'
                
            }
        }

        stage('Image Scan') {
            steps {
      	        sh ' trivy image --format template --template "@/usr/local/share/trivy/templates/html.tpl" -o report.html ${IMAGE_REPO}/${NAME}:${VERSION} '
            }
        }

        stage('Upload Scan report to AWS S3') {
              steps {
                  sh 'aws s3 cp report.html s3://devsecops-project/'
              }
         }

        stage('Docker  Push') {
            steps {
                withVault(configuration: [skipSslVerification: true, timeout: 60, vaultCredentialId: 'vault-cred', vaultUrl: 'http://your-vault-server-ip:8200'], vaultSecrets: [[path: 'secrets/creds/docker', secretValues: [[vaultKey: 'username'], [vaultKey: 'password']]]]) {
                    sh "docker login -u ${username} -p ${password} "
                    sh 'docker push ${IMAGE_REPO}/${NAME}:${VERSION}'
                    sh 'docker rmi ${IMAGE_REPO}/${NAME}:${VERSION}'
                }
            }
        }
        stage('Clone/Pull Repo') {
            steps {
                script {
                    if (fileExists('gitops-argocd')) {

                        echo 'Cloned repo already exists - Pulling latest changes'

                        dir("gitops-argocd") {
                          sh 'git pull'
                        }

                    } else {
                        echo 'Repo does not exists - Cloning the repo'
                        sh 'git clone -b feature https://github.com/praveensirvi1212/DevOps_MasterPiece-CD-with-argocd.git'
                    }
                }
            }
        }        
        
        stage('Update Manifest') {
            steps {
                dir("gitops-argocd/jenkins-demo") {
                    sh 'sed -i "s#siddharth67.*#${IMAGE_REPO}/${NAME}:${VERSION}#g" deployment.yaml'
                    sh 'cat deployment.yaml'
                }
            }
        }

        stage('Commit & Push') {
            steps {
                dir("gitops-argocd/jenkins-demo") {
                    sh "git config --global user.email 'praveen@gmail.com'"
                    sh 'git remote set-url origin https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}'
                    sh 'git checkout feature'
                    sh 'git add -A'
                    sh 'git commit -am "Updated image version for Build - $VERSION"'
                    sh 'git push origin feature'
                }
            }
        } 

        stage('Raise PR') {
            steps {
                sh 'echo ${GITHUB_TOKEN} | gh auth login --with-token'
                sh "gh pr create -t 'image tag updated' -b 'check and merge it' "
            }
        }               
    }

    post{
        always{
            sendSlackNotifcation()
            }
        }
}


def sendSlackNotifcation()
{
    if ( currentBuild.currentResult == "SUCCESS" ) {
        buildSummary = "Job_name: ${env.JOB_NAME}\n Build_id: ${env.BUILD_ID} \n Status: *SUCCESS*\n Build_url: ${BUILD_URL}\n Job_url: ${JOB_URL} \n"
        slackSend( channel: "#devops", token: 'slack-token', color: 'good', message: "${buildSummary}")
    }
    else {
        buildSummary = "Job_name: ${env.JOB_NAME}\n Build_id: ${env.BUILD_ID} \n Status: *FAILURE*\n Build_url: ${BUILD_URL}\n Job_url: ${JOB_URL}\n  \n "
        slackSend( channel: "#devops", token: 'slack-token', color : "danger", message: "${buildSummary}")
    }
}


