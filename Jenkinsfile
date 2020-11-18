pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        sh'''
            echo 'FROM debian:latest' > Dockerfile
            echo 'CMD ["/bin/echo", "HELLO WORLD...."]' >> Dockerfile
        '''
        script{
            docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-user') {
                      def image = docker.build('iamunrootable/helloworld:latest')
                      image.push()        
                }
        
            }
        }
    }

    stage('Lint HTML') {
      steps {
        sh 'tidy -q -e *.html'
      }
    }
    
    stage('Security Scan') {
      steps {
        writeFile file: 'anchore_images', text: image
        anchore name: 'anchore_images', bailOnFail: false, engineRetries:'1800'
        }
    }
  

    stage('Upload to AWS') {
      steps {
        withAWS(region: 'us-west-2', credentials:'aws-static') {
          sh 'echo "Uploading content with AWS creds"'
          s3Upload(pathStyleAccessEnabled: true, payloadSigningEnabled: true, file: 'index.html', bucket: 'static-jenkins-pipeline')
        }

      }
    }

  }
}
