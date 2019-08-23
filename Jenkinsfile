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
            
             withSonarQubeEnv('SONAR 6.4') {
                    sh "${scannerHome}/bin/sonar-scanner"
                    sh "cat .scannerwork/report-task.txt"
                    def props = readProperties  file: '.scannerwork/report-task.txt'
                    echo "properties=${props}"
                    def sonarServerUrl=props['serverUrl']
                    def ceTaskUrl= props['ceTaskUrl']
                    def ceTask
                    timeout(time: 1, unit: 'MINUTES') {
                        waitUntil {
                            def response = httpRequest ceTaskUrl
                            ceTask = readJSON text: response.content
                            echo ceTask.toString()
                            return "SUCCESS".equals(ceTask["task"]["status"])
                        }
                    }
                    def response2 = httpRequest url : sonarServerUrl + "/api/qualitygates/project_status?analysisId=" + ceTask["task"]["analysisId"], authentication: 'jenkins_scanner'
                    def qualitygate =  readJSON text: response2.content
                    echo qualitygate.toString()
                    if ("ERROR".equals(qualitygate["projectStatus"]["status"])) {
                        error  "Quality Gate failure"
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
