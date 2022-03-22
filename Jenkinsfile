def globalStackName="cfAnsible"
def gloablKeyPairName="ansible"

pipeline {
  agent {
    kubernetes {
      label "aws-build"
      yamlFile 'build.yaml'
    }
  }
  stages {
    stage('Create the Environment') {
      steps {
        container(name: 'aws', shell: '/bin/bash') {
          withCredentials([[
            $class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: "ansible",
            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
            ]]) {
                sh """
                export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                export AWS_DEFAULT_REGION=us-east-1

                aws --version
                aws iam get-user

                #generation of key pair
                aws ec2 create-key-pair --key-name ${gloablKeyPairName} --query 'KeyMaterial' --output text > ${gloablKeyPairName}.pem
                chmod 400 ${gloablKeyPairName}.pem

                cp ./aws/setup-env.yaml setup-env.yaml

                aws cloudformation create-stack --stack-name ${globalStackName} --template-body file://setup-env.yaml  --parameters ParameterKey=NameOfService,ParameterValue=ansibleServiceStack ParameterKey=KeyName,ParameterValue=${gloablKeyPairName} --query 'StackId' --output text > stackId.txt

                stackId=`cat stackId.txt`

                stackStatus=CREATE_IN_PROGRESS

                while [ \$stackStatus == "CREATE_IN_PROGRESS" ]
                do
                  sleep 15

                  aws cloudformation describe-stacks --stack-name ${globalStackName} --query 'Stacks[0].StackStatus' --output text > stackStatus.txt
                  cat stackStatus.txt
                  stackStatus=`cat stackStatus.txt`
                  
                done

                aws cloudformation describe-stacks --stack-name ${globalStackName}

                aws cloudformation describe-stacks --stack-name ${globalStackName} --query 'Stacks[0].Outputs[0].OutputValue' --output text > web1.txt
                aws cloudformation describe-stacks --stack-name ${globalStackName} --query 'Stacks[0].Outputs[1].OutputValue' --output text > web2.txt
                aws cloudformation describe-stacks --stack-name ${globalStackName} --query 'Stacks[0].Outputs[2].OutputValue' --output text > lb.txt
                aws cloudformation describe-stacks --stack-name ${globalStackName} --query 'Stacks[0].Outputs[3].OutputValue' --output text > dns.txt
                
                cat web1.txt
                cat web2.txt
                cat lb.txt
                cat dns.txt


                #aws ec2 describe-instances
                
              """
          }
        }
      }
    }
    stage('generate control and setup control files') {
      steps {
        container(name: 'ansible', shell: '/bin/bash') {
          sh """
            app1=`cat web1.txt`
            app2=`cat web2.txt`
            lb=`cat lb.txt`

`cat <<EOT >> hosts-dev
[webservers]
app1 ansible_host=\$app1
app2 ansible_host=\$app2

[loadbalancers]
lb1 ansible_host=\$lb

[local]
control ansible_connection=local
EOT`

            cp ${gloablKeyPairName}.pem ./ansible/${gloablKeyPairName}.pem
            cp hosts-dev ./ansible/hosts-dev

            cd ./ansible
            ls -lrt
            cat hosts-dev

            #ssh -y -i ec2-user@\$app1
            #whoami
            #exit

          """
        }
      }
    }
    stage('Setup Environment') {
      steps {
        container(name: 'ansible', shell: '/bin/bash') {
          sh """
            pwd
            cd ./ansible
            ls -lrt

            chmod 400 ${gloablKeyPairName}.pem

            ansible-playbook playbooks/all-playbooks.yml
          """
        }
      }
    }
  }
}
