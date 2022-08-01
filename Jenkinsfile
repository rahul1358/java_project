pipeline {
    agent any
    tools {
       maven 'M2_HOME'
    }
    environment {
        registry = '669404210120.dkr.ecr.us-east-1.amazonaws.com/imgrepo'
        resistryCredentials = 'ecr-login'
        dockerImage = ''
    }
    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
                echo 'hi'
            }
        }
        stage ('checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/rahul1358/java_project.git']]])
            }
        }
        stage('Build') {
           steps {
                sh 'mvn clean package  -DskipTests'
           }
        }
        stage('sonarqube checks') {
            steps {
                script {
        //        withSonarQubeEnv(installationName: 'Sonarscanner', credentialsId: 'SonarCloud') {
                  // withSonarQubeEnv(credentialsId: 'sonarqube-2', installationName: 'Sonarscanner_2')
                    withSonarQubeEnv(credentialsId: 'sonarnew', installationName: 'sonaranalysis'){
                 sh 'mvn clean package sonar:sonar'
                 }
                  timeout(time: 3, unit: 'MINUTES') {
                     def qg = waitForQualityGate()
                  if (qg.status != 'OK') {
                     error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                }
           }
        }
   }
        stage('Build the Image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }   
        }
      stage('Deploy the image to Amazon ECR') {
            steps {
                script {
                docker.withRegistry("http://" + registry, "ecr:us-east-1:" + resistryCredentials) {
                dockerImage.push()
         }
        }
      }
    }
  }
   post { 
   success { 
          mail bcc: '', body: 'Pipeline Build is Successfully Executed', cc:'', from: 'kalakotarahul@gmail.com', replyTo: 'rahulroy1358@gmail.com', subject: ' The Pipeline is Success', to: 'rahulroy1358@gmail.com' 
        }
        failure {
            mail bcc: '', body: 'Pipeline Build is Failed while Executing', cc:'', from: 'kalakotarahul@gmail.com', replyTo: 'rahulroy1358@gmail.com', subject: ' The Pipeline is Failure', to: 'devopsuser12@gmail.com'
        }
    }
}
