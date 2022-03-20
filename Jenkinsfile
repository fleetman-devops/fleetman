pipeline {
   agent any
   stage('Checkout') { 
      checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/dashashutosh80/fleetman-webapp']]])
   }
   stage('Build') {
      sh "mvn -DskipTests clean package"
   }
   stage('Archive') {
      archiveArtifacts artifacts: 'target/*.war', followSymlinks: false
   }
}
