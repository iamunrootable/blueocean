#!/usr/bin/env groovy
pipeline{
    agent {
        docker {
            image 'docker:stable'
        }
    }
    environment {
        IMAGE_NAME = 'iamunrootable/helloworld'
        IMAGE_TAG = 'latest'
    }
    stages {
        stage('Build Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:ci .'
            }
        }
        stage('Scan') {
            steps {        
                sh 'apk add bash curl'
                sh 'curl -s https://ci-tools.anchore.io/inline_scan-v0.6.0 | bash -s -- -d Dockerfile -b .anchore_policy.json ${IMAGE_NAME}:ci'
            }
        }
        stage('Push Image') {
            steps {
                withDockerRegistry([credentialsId: "dockerhub-user", url: "https://index.docker.io/v1/"]){
                    sh 'docker tag ${IMAGE_NAME}:ci ${IMAGE_NAME}:${IMAGE_TAG}'
                    sh 'docker push ${IMAGE_NAME}:${IMAGE_TAG}'
                }
            }
        }
    }
}
