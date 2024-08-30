pipeline {
       agent any

       tools {
           maven 'maven3'
       }

       environment {
           SLACK_CHANNEL = '#appdevops'
       }

       stages {
           stage('Checkout') {
               steps {
                   git branch: 'main', url: 'https://github.com/DevandMlOps/calc-junit'
               }
           }

           stage('Build') {
               steps {
                   sh 'mvn clean package'
               }
           }

           stage('Test') {
               steps {
                   sh 'mvn test'
               }
           }           
       }

       post {
           always {
               junit '**/target/surefire-reports/*.xml'
               
               script {
                   def COLOR_MAP = [
                       'SUCCESS': 'good',
                       'FAILURE': 'danger',
                       'UNSTABLE': 'warning',
                       'ABORTED': 'warning'
                   ]
                   
                   def buildStatus = currentBuild.currentResult
                   def errorMessage = ""
                   
                   if (buildStatus == 'FAILURE') {
                       errorMessage = currentBuild.rawBuild.getLog(1000).join('\n')
                       if (errorMessage.length() > 1000) {
                           errorMessage = errorMessage.take(1000) + "... [mensaje truncado]"
                       }
                   }
                   
                   slackSend(
                       channel: env.SLACK_CHANNEL,
                       color: COLOR_MAP[buildStatus],
                       message: """*${buildStatus}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}*
                           Branch: ${env.GIT_BRANCH}
                           Commit: ${env.GIT_COMMIT}
                           Author: ${env.GIT_AUTHOR_NAME}
                           More Info at: ${env.BUILD_URL}
                           ${buildStatus == 'FAILURE' ? "\nError Details:\n```${errorMessage}```" : ''}
                       """.stripIndent()
                   )
               }
           }
       }
   }
