pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar'
        DOCKERHUB_CREDENTIALS=credentials('docker-cred')
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Frawatson/amazonclone.git'
            }
        }
        stage('Sonar Scan') {
            steps{
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=amazonclone \
                    -Dsonar.projectKey=sqp_ffd9caa2283376495bfc801702cf8d0b135170e2 '''
                }
            }
        }
         stage('Install Dependecies') {
            steps{
                sh "npm install"
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker'){   
                       sh "docker build -t amazon-clone ."
                       sh "docker tag amazon-clone frawatson/amazon-clone:latest "
                       sh "docker push frawatson/amazon-clone:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image frawatson/amazon-clone:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name amazon-clone -p 80:80 frawatson/amazon-clone:latest'
            }
        }
    }
}
