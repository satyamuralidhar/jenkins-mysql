pipeline {
    agent any
    environment{

        dockerImage = ''

        registryurl = 'muralidhar123/dbmysql'

        dockercred = 'dockerhub' 
    }

    stages {
        stage('cleanWorkspace'){
            steps{
                step([$class: 'WsCleanup'])
            }
        }    
        stage("scm using git"){
        
            steps {
                git url: 'https://github.com/satyamuralidhar/mysql-docker-pipeline.git'
            }
        }
           
        stage('Building image') {
            steps{
                script {
                    dockerImage = docker.build registryurl + ":v$BUILD_NUMBER"
                }
            }
        }
        stage('Deploy Image') {
            steps{
                script {
                    docker.withRegistry( '', dockercred ) {
                        dockerImage.push()
                    }   
                }
            }
        }
        stage('Remove Unused docker image') {
            steps{
                sh "docker rmi -f $registryurl:v$BUILD_NUMBER"
            }
        }
        stage("git repo") {
            steps {
              sh 'git clone https://github.com/satyamuralidhar/mysql-helm.git'
              sh 'cd mysql-helm'
              sh 'ls -la'
              sh "sed -i '/tag/s/:.*\$/: v${BUILD_NUMBER}/g' mysql-helm/mysql/values.yaml"
              //sh 'pwd && helm upgrade mysql --install mysql-helm/mysql'
              sh label: '', script: 'cd /var/lib/jenkins/workspace/mysql-helm/'
              sh 'helm upgrade --install --force mysql mysql-helm/mysql'  
              
                        //sh 'helm install mysql --generate-name'
              //sh 'helm install --name-template mysql mysql && helm repo update && helm upgrade mysql --install mysql'        
                
            }
        }
    }
       
    // slack notification  
    post {
        success {
            slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")

        }
        failure {
            slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        } 
    }        
         
} 
