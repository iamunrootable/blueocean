#!/usr/bin/env groovy
pipeline{
    agent any
    environment {
        IMAGE_NAME = 'iamunrootable/helloworld'
        IMAGE_TAG = 'latest'
    }
    stages {
        stage('Build Image') {
            steps {
                sh'''
                echo 'FROM debian:latest' > Dockerfile
                echo 'CMD ["/bin/echo", "HELLO WORLD...."]' >> Dockerfile
                docker build -t ${IMAGE_NAME}:ci .
                '''
            }
        }
        stage('Scan Image') {
            steps {                     
                sh 'curl -s https://ci-tools.anchore.io/inline_scan-v0.6.0 | bash -s -- -d Dockerfile -b ./policy_bundle.json ${IMAGE_NAME}:ci'
            }
        }
        stage('Push Image to Registry') {
            steps {
                withDockerRegistry([credentialsId: "dockerhub-user", url: "https://index.docker.io/v1/"]){
                    sh 'docker tag ${IMAGE_NAME}:ci ${IMAGE_NAME}:${IMAGE_TAG}'
                    sh 'docker push ${IMAGE_NAME}:${IMAGE_TAG}'
                }
            }
        }
        stage('Upload Image to AWS') {
              steps {
                  withAWS(region:'us-west-2',credentials:'aws-static') {
                  sh 'echo "Uploading content with AWS creds"'
                      s3Upload(pathStyleAccessEnabled: true, payloadSigningEnabled: true, file:'index.html', bucket:'static-jenkins-pipeline-acloudlabs')
                  }
              }
         }
         stage('Prune Images') {
            steps {
                sh 'docker image prune --all -f'
            }
        }        
    }
}
