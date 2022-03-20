pipeline {
   agent any
   environment {
        AWS_ACCOUNT_ID="542591410366"
        AWS_DEFAULT_REGION="us-east-1" 
        IMAGE_REPO_NAME="fleetman"
        registry = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
   }
   stages {
      stage('Checkout') { 
         steps{
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/dashashutosh80/fleetman-webapp']]])
         }
      }
      stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
      }
      // Building Docker images
      stage('Building image') {
         steps{
            script {
               dockerImage = docker.build registry + ":latest"
            }
         }
      }
      // Uploading Docker images into AWS ECR
      stage('Pushing to ECR') {
         steps{  
               script {
                     sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                     sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:latest"
               }
         }
      }
   }
}
