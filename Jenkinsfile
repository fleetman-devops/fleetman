pipeline {
   agent any
   stages {
      stage('Checkout') { 
         steps{
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/dashashutosh80/fleetman-webapp']]])
         }
      }
      stage('Build') {
         steps{
            sh "mvn -DskipTests clean package"
         }
      }
      stage('Archive') {
         steps{
            echo "Archiving data in S3"
            archiveArtifacts artifacts: 'target/*.war', followSymlinks: false
         }
      }
   }
}
