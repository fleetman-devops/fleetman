pipeline {
   agent any
   // tools {
   //  maven 'maven3'
   // }
   environment {
        AWS_ACCOUNT_ID="542591410366"
        AWS_DEFAULT_REGION="us-east-1" 
        IMAGE_REPO_NAME="fleetman"
        registry = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
   }
   stages {
      // stage('Log in to AWS ECR') {
      //    steps {
      //       script {
      //          sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
      //       }
      //    }
      // }
      stage('Checkout') { 
         steps{
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/dashashutosh80/fleetman-webapp']]])
         }
      }
      stage('Build'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            // Post Successful Build Steps
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
      }
      // Building Docker Images
      stage('Building image') {
         steps{
            script {
               dockerImage = docker.build registry + ":latest"
               //dockerImage = docker.build registry + "${env.BUILD_NUMBER}"
            }
         }
      }
      // Uploading Docker Images into AWS ECR
      // stage('Deploy to ECR: Type 1') {
      //    steps{  
      //          script {
      //                sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:latest"
      //          }
      //    }
      // }
      stage('Deploy to ECR: Type 2') {
         steps {
            script {
            // This step should not normally be used in your script. Consult the inline help for details.
               withDockerRegistry(credentialsId: 'ecr:us-east-1:aws_credentials', url: 'https://542591410366.dkr.ecr.us-east-1.amazonaws.com/fleetman') {
                  dockerImage.push("latest")
               }
            }
         }
      }
      //This step removes old images from the instance where Docker images are being built.
      stage('Remove local images') {
         steps {
            script {
                sh "docker rmi -f ${registry}:latest"
            }
         }
      }
   }
}
