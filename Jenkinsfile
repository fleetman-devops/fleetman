pipeline {
   agent any
   environment {
        AWS_ACCOUNT_ID="542591410366"
        AWS_DEFAULT_REGION="us-east-1" 
        IMAGE_REPO_NAME="fleetman"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
   }
   stages {
      stage('Logging into AWS ECR') {
            steps {
                script {
                  sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
               }  
            }
      }
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
               dockerImage = docker.build "${IMAGE_REPO_NAME}:${env.BUILD_NUMBER}"
               }
         }
      }
      // Uploading Docker images into AWS ECR
      stage('Pushing to ECR') {
         steps{  
               script {
                     //sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                     sh "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${env.BUILD_NUMBER}"
               }
         }
      }
   }
}
