pipeline{
    agent any
    tools {
      maven 'maven-3'
      terraform 'terraform'
    }
    environment {
      DOCKER_TAG = getVersion()
    }
    
    stages{
        stage('SCM'){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/jaymezon/docker-ansible-jenkins'
            }
        }
        stage('Maven Build'){
            steps{
                sh "mvn clean package"
            }
        }
        
        stage('Compile-Package'){
            // Get maven home path
            steps{
                 // Get maven home path
                def mvnHome =  tool name: 'maven-3', type: 'maven'   
                sh "${mvnHome}/bin/mvn package"
            }
        } 
        stage('SonarQube Analysis') {
            steps{
                    def mvnHome =  tool name: 'maven-3', type: 'maven'
                    withSonarQubeEnv('sonar-8') { 
                    sh "${mvnHome}/bin/mvn sonar:sonar"
                    }
                }
         }
        stage('Compile-Package') {
            steps{
                timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                          error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                  } 
                }
            }
        
        // stage("Quality Gate Statuc Check"){
        //     steps{
        //         timeout(time: 1, unit: 'HOURS') {
        //             def qg = waitForQualityGate()
        //             if (qg.status != 'OK') {
        //                 slackSend baseUrl: 'https://hooks.slack.com/services/',
        //                 channel: '#jenkins-pipeline-demo',
        //                 color: 'danger', 
        //                 message: 'SonarQube Analysis Failed', 
        //                 teamDomain: 'jaymezon',
        //                 tokenCredentialId: 'slack-demo'
        //                 error "Pipeline aborted due to quality gate failure: ${qg.status}"
        //             }
        //         }
        //     }
        // }   
        stage ("terraform init") {
            steps {
                sh 'terraform init'
            }
        }
        stage ("terraform fmt") {
            steps {
                sh 'terraform fmt'
            }
        }
        stage ("terraform validate") {
            steps {
                sh 'terraform validate'
            }
        }
        stage ("terrafrom plan") {
            steps {
                sh 'terraform plan '
            }
        }
        // stage ("terraform apply") {
        //     steps {
        //         sh 'terraform apply --auto-approve'
        //     }
        // } https://www.coachdevops.com/2021/07/jenkins-terraform-integration-how-do.html
        stage ("terraform Action") {
            steps {
                echo "Terraform action is --> ${action}"
                sh ('terraform ${action} --auto-approve') 
           }
        } 
        stage('Docker Build'){
            steps{
                sh "docker build . -t jaymezon/sembeapp:${DOCKER_TAG} "
            }
        }
        
        stage('DockerHub Push Image'){
            steps{
                // login to dockerhub account and push image
                withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerHubPwd')]) {
                    sh "docker login -u jaymezon -p ${dockerHubPwd}"
                }
                
                sh "docker push jaymezon/sembeapp:${DOCKER_TAG} "
            }
        }
        
        stage('Docker Deploy'){
            steps{
              ansiblePlaybook credentialsId: 'dev-server', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}",
              installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
            }
        }
//         stage('Email Notification'){
//             steps{
//                 mail bcc: '', body: '''Hi Welcome to jenkins email alerts
//                 Thanks
//                 Jay''', cc: '', from: '', replyTo: '', subject: 'Jenkins Job', to: 'jaymezon@gmail.com'
//             }
//         }
        
    }
}

def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
