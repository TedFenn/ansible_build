pipeline {
  agent {
    kubernetes {
      label "aws-build"
      yamlFile 'build.yaml'
    }
  }
  stages {
    stage('AWS Image') {
      steps {
        container(name: 'aws', shell: '/bin/bash') {
          sh '''#!/busybox/sh
            echo 'hello'
          '''
        }
      }
    }
  } 
}
