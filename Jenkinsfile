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
  node{
    def imageLine = 'iamunrootable/helloworld:latest' + ' ' + env.WORKSPACE + '/DockerFile'

    stage('Security Scan') {
      steps {
        writeFile file: 'anchore_images', text: imageLine
        anchore name: 'anchore_images', policyName: 'anchore_policy', bailOnFail: false, inputQueries: [[query: 'list-packages all'], [query: 'cve-scan all']]
        }
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
