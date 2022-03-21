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
          withAWS(credentials: 'ansible', region: 'us-east-1') {
            sh '''
              aws --version
              aws iam get-user
              
              aws ec2 create-key-pair --key-name ansibleKey
              
              ls -l
              ls -l /home
            '''
          }
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
