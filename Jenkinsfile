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
            // If in the archiving stage, a broken pipe error occurs check the S3 bucket configuration such as bucket name, region, etc
            // under manage jenkins > AWS > Amazon S3 Bucket Access settings
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
               dockerImage = docker.build registry + ":${env.BUILD_NUMBER}" 
            }
         }
      }
      // Replace image key value with placeholders for spinnaker parameters.
      // This stage must come before uploading to S3 as spinnaker needs to pull this modified manifest from S3
      stage('Parameterize Manifest') {
         steps{
            script {
               sh '''
                     #!/bin/bash
                     cat deploy.yaml | grep image
                     sed -i 's|image: .*|image: "${ parameters.image }"|' deploy.yaml
                     cat deploy.yaml | grep image
                  '''
            }
         }
      }
      // Upload K8s manifest to S3.
      // This stage requires a S3 profile to be configured beforehand in jenkins. Under Manage Jenkins > Configure System > Amazon S3 Profiles: give any name to profile and provide access key and secret access key
      stage('Upload manifest to S3'){
         steps{
               // Generate syntax with s3Upload: publish artifacts to S3 bucket option from pipeline syntax generator
               sh 'echo "Uploading manifests to S3"'
               s3Upload consoleLogLevel: 'INFO', dontSetBuildResultOnFailure: false, dontWaitForConcurrentBuildCompletion: false, entries: [[bucket: 'jenkins-manifests', excludedFile: '', flatten: false, gzipFiles: false, keepForever: false, managedArtifacts: false, noUploadOnFailure: false, selectedRegion: 'us-west-2', showDirectlyInBrowser: false, sourceFile: '*.yaml', storageClass: 'STANDARD', uploadFromSlave: false, useServerSideEncryption: false]], pluginFailureResultConstraint: 'FAILURE', profileName: 'fleetman', userMetadata: []
            }
      }
      //Uploading Docker Images into AWS ECR
      stage('Deploy to ECR') {
         steps{
               script {
                  sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${env.BUILD_NUMBER}"
               }
         }
      }
      // stage('Deploy Image to ECR: Type 2') {
      //    steps {
      //       script {
      //       // This step should not normally be used in your script. Consult the inline help for details.
      //          withDockerRegistry(credentialsId: 'ecr:us-west-2:aws_credentials', url: 'https://542591410366.dkr.ecr.us-west-2.amazonaws.com/fleetman') {
      //             dockerImage.push("${env.BUILD_NUMBER}")
      //          }
      //       }
      //    }
      // }
      stage('Trigger spinnaker webhook'){
         steps {
            script {
               // Use """ to escape double quotes
               sh """
                  #!/bin/bash
                  echo "Triggering Spinnaker"
                  curl http://a0d8e4961e0c748aeb3c611e108ab421-297111095.us-west-2.elb.amazonaws.com/webhooks/webhook/spinnaker -X POST -H "content-type: application/json" -d '{"parameters": {"image":"542591410366.dkr.ecr.us-west-2.amazonaws.com/fleetman:${env.BUILD_NUMBER}"}}'
                  """
            }
         } 
      }
      //This step removes old images from the instance where Docker images are being built.
      stage('Remove local images') {
         steps {
            script {
                sh "docker rmi -f ${registry}:${env.BUILD_NUMBER}"
            }
         }
      }
   }
}
