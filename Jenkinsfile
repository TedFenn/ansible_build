pipeline {
  agent {
    kubernetes {
      label "aws-build"
      yamlFile 'build.yaml'
    }
  }
  stages {
    stage('Configure Environment') {
      steps {
        container(name: 'aws', shell: '/bin/bash') {
          sh '''
            aws --version
          '''
        }
      }
    }
    stage('Setup Environment') {
      steps {
        container(name: 'ansible', shell: '/bin/bash') {
          sh '''
            python -m pip --version
          '''
        }
      }
    }
  } 
}
