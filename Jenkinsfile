pipeline {
    agent any
    
    tools {nodejs "nodejs"}
   
    
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
                
            }
        }
    

       stage('Test') {
            steps {
                sh 'npm run test'
                sh 'npm test -- --coverage'
            }
        }
         stage ('zipping'){
            steps {
               
                sh 'zip -r build.zip ./build'
            }
        }
       
      
       stage('Sonarqube analysis') {
               environment {
                 scannerHome=tool 'sonar scanner'
            }
            
              steps { 
                   withSonarQubeEnv('Sonar') {
                  
              sh 'npm test -- --coverage'
              sh '${scannerHome}/bin/sonar-scanner -Dproject.settings=./sonar.properties'
                  }
              }
        }
       stage("Quality Gate") {
            steps {
               // qualityGateUrl=18.224.155.110:9000/quality_gates/show/116
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
       }
    

        stage ('Uploading artifact to nexus'){
        steps{
        withCredentials([usernamePassword(credentialsId: 'arko489_nexus', passwordVariable: 'pwd', usernameVariable: 'usr')]) {
         sh 'curl -v -u $usr:$pwd --upload-file build.zip http://18.224.155.110:8081/nexus/content/repositories/devopstraining/arko/build.zip'
            }
        }
       }
    }
        post {
    success {
      slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }
    failure {
      slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }
  }
}
