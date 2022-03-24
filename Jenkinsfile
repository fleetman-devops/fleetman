pipeline {
   agent any
   // tools {
   //  maven 'maven3'  
   // }
   environment {
        AWS_ACCOUNT_ID="542591410366"
        AWS_DEFAULT_REGION="us-west-2" 
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
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/fleetman-devops/fleetman']]])
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
               dockerImage = docker.build registry + "${env.BUILD_NUMBER}" 
               //dockerImage = docker.build registry + "${env.BUILD_NUMBER}"
            }
         }
      }
      stage('Parameterize Manifest') {
         steps{
            script {
               sh '''
                     #!/bin/bash
                     cat deploy.yml | grep image
                     sed -i 's|image: .*|image: "${ parameters.image }"|' deploy.yml
                     cat deploy.yml | grep image
                  '''
            }
         }
      }
      stage('Upload manifest to S3'){
         steps{
            withAWS(region:'us-west-2',credentials:'aws_credentials') {
                  sh 'echo "Uploading manifests to S3"'
                  s3Upload consoleLogLevel: 'INFO', dontSetBuildResultOnFailure: false, dontWaitForConcurrentBuildCompletion: false, entries: [[bucket: 'jenkins-manifests', excludedFile: '', flatten: false, gzipFiles: false, keepForever: false, managedArtifacts: false, noUploadOnFailure: false, selectedRegion: 'us-west-2', showDirectlyInBrowser: false, sourceFile: '*.yaml', storageClass: 'STANDARD', uploadFromSlave: false, useServerSideEncryption: false]], pluginFailureResultConstraint: 'FAILURE', profileName: 'fleetman', userMetadata: []
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
      stage('Deploy Image to ECR: Type 2') {
         steps {
            script {
            // This step should not normally be used in your script. Consult the inline help for details.
               withDockerRegistry(credentialsId: 'ecr:us-west-2:aws_credentials', url: 'https://542591410366.dkr.ecr.us-west-2.amazonaws.com/fleetman') {
                  dockerImage.push("${env.BUILD_NUMBER}")
               }
            }
         }
      }
      stage('Trigger spinnaker webhook'){
         steps {
            script {
               sh '''
                  #!/bin/bash
                  echo "Triggering Spinnaker"
                  echo '{"parameters": {"image":"542591410366.dkr.ecr.us-west-2.amazonaws.com/fleetman:${env.BUILD_NUMBER}"}}'
                  curl https://ptsv2.com/t/pmmth-1648089958/post -X POST -H "content-type: application/json" -d '{"parameters": {"image":"542591410366.dkr.ecr.us-west-2.amazonaws.com/fleetman:${env.BUILD_NUMBER}"}}'
                  '''
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
