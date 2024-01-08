node{
    
def mavenHome = tool name: 'maven 3.9.6'

echo "Job name is: ${env.JOB_NAME}"
echo "Node name is: ${env. NODE_NAME}"
echo "Jenkins home is: ${env. JENKINS_HOME}"
echo "Jenkins URL is: ${env. JENKINS_URL}"


properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')), [$class: 'JobLocalConfiguration', changeReasonComment: ''], pipelineTriggers([pollSCM('* * * * *')])])


try{
stage('checkoutcode'){
    sendSlacknotifications("STARTED")
git branch: 'development', credentialsId: '5ac199a6-aa64-4e99-ba01-747ddcbb33b6', url: 'https://github.com/rajumanne98/maven-web-application.git'
}

stage ('build'){
sh "${mavenHome}/bin/mvn clean package"
}

stage('Executesonarqubereport'){
sh "${mavenHome}/bin/mvn clean sonar:sonar"
}

stage('uploadArtifactIntoNexus'){
sh "${mavenHome}/bin/mvn clean deploy"
}

stage('DeployappintoTomcatserver'){
sshagent(['cf92bf7b-36df-4e8d-b379-de8d20f567f8']) {
sh "scp -o strictHostkeychecking=no target/maven-web-application.war ec2-user@172.31.18.194:/opt/apache-tomcat-9.0.83/webapps/"
}
}

}
catch(e){
currentBuild.result = "FAILURE"
    throw e
}
finally{
sendSlacknotifications(currentBuild.result)
}
}//node closing


def sendSlacknotifications(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESSFUL'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESSFUL') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    colorName = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary)
}

