import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles) {
    if (p['publishMethod'] == 'FTP') {
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
    }
  }
  error("No FTP publish profile found")
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=233ea338-62e4-4371-b676-6b1fb2448653',
            'AZURE_TENANT_ID=672423e6-17b3-4280-801a-71b8b38bdd38'])  {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName = 'jenkinsapp'

      // Login to Azure
      withCredentials([usernamePassword(credentialsId: '99d2dade-fcc5-4ed9-80fa-60ccadf1fb42', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
        sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }

      // Get publish settings
      def pubProfilesJson = sh(script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName --query '[].{publishUrl: publishUrl, userName: userName, userPWD: userPWD, publishMethod: publishMethod}' -o json", returnStdout: true)
      echo "Publishing profiles: ${pubProfilesJson}"
      
      def ftpProfile = getFtpPublishProfile(pubProfilesJson)
      echo "FTP Profile: ${ftpProfile}"

      // Use FTP for deployment
      sh """
        curl -T target/calculator-1.0.war --user ${ftpProfile.username}:${ftpProfile.password} ${ftpProfile.url}
      """

      // Logout
      sh 'az logout'
    }
  }
}
