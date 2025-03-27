import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=501f9291-1a9f-4cb0-93d5-0aa60df35b4d',
        'AZURE_TENANT_ID=5837c53c-f27a-4ff2-ad09-d70f9608717c']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName = 'jenkins-sample-app123'
      // login Azure
      withCredentials([usernamePassword(
        credentialsId: 'AzureServicePrincipal', 
        passwordVariable: 'AZURE_CLIENT_SECRET', 
        usernameVariable: 'AZURE_CLIENT_ID'
      )]) {
       sh """
az login --service-principal -u \$AZURE_CLIENT_ID -p \$AZURE_CLIENT_SECRET -t \$AZURE_TENANT_ID
az account set -s \$AZURE_SUBSCRIPTION_ID

pubProfilesJson=\$(az webapp deployment list-publishing-profiles -g ${resourceGroup} -n ${webAppName} --output json)

url=\$(echo "\$pubProfilesJson" | jq -r '.[] | select(.publishMethod=="FTP") | .publishUrl')
username=\$(echo "\$pubProfilesJson" | jq -r '.[] | select(.publishMethod=="FTP") | .userName')
password=\$(echo "\$pubProfilesJson" | jq -r '.[] | select(.publishMethod=="FTP") | .userPWD')

echo "URL: \$url"
echo "Username: \$username"

curl -T target/calculator-1.0.war ftp://\$url/site/wwwroot/webapps/ROOT.war --user \$username:\$password

az logout
"""


      }
    }
  }
}
