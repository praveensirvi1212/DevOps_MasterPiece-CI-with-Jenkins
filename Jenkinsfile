pipeline {
    agent any

    environment {
        NAME = "spring-app"
        VERSION = "${env.BUILD_ID}"
        GIT_COMMIT = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
        IMAGE_REPO = "praveensirvi"
        GIT_REPO_NAME = "DevOps_MasterPiece-CD-with-argocd"
        GIT_USER_NAME = "praveensirvi1212"
       
    }

    tools { 
        maven 'maven-3.8.6' 
    }
    stages {
        stage('Checkout git') {
            steps {
              git branch: 'main', url:'https://github.com/praveensirvi1212/DevOps_MasterPiece-CI-with-Jenkins.git'
            }
        }
        
        stage ('Build & JUnit Test') {
            steps {
                sh 'mvn clean install' 
            }
        }

}



