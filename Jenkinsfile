def globalStackName="cfAnsible3"
def gloablKeyPairName="ansible3"

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
          withCredentials([[
            $class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: "ansible",
            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
            ]]) {
                sh """
                export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                export AWS_DEFAULT_REGION=us-east-1

                aws --version
                aws iam get-user

                #generation of key pair
                aws ec2 create-key-pair --key-name ${gloablKeyPairName} --query 'KeyMaterial' --output text > ${gloablKeyPairName}.pem
                chmod 640 ${gloablKeyPairName}.pem

                cp ./aws/setup-env.yaml setup-env.yaml

                aws cloudformation create-stack --stack-name ${globalStackName} --template-body file://setup-env.yaml  --parameters ParameterKey=NameOfService,ParameterValue=ansibleServiceStack ParameterKey=KeyName,ParameterValue=ansible --query 'StackId' --output text > stackId.txt

                stackId=`cat stackId.txt`
                
                aws cloudformation describe-stacks --stack-name ${globalStackName}

                ls -l
                
              """
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
