pipeline {
  agent any
  parameters {
        string(name: 'AWS_DEFAULT_REGION', defaultValue: 'us-east-1', description: 'The region to deploy to')
  }

  stages {
    stage('checkout'){
        steps{
            git scm
            sh "pwd;ls -lat"
        }
    }
    stage('create stack') {
        steps {
            withCredentials([[
                $class: 'AmazonWebServicesCredentialsBinding',
                credentialsId: 'awstoken',
                accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
            ]]) {             
                  sh '''#!/usr/bin/env bash
                    env | sort -u
                    aws ec2 describe-instances
                    if aws cloudformation describe-stacks  --query 'Stacks[].StackName' | grep jenkinsHello ; then
                      aws cloudformation update-stack  --stack-name jenkinsHello --template-body file://2-ec2.yaml  --parameters ParameterKey=InstanceType,ParameterValue=t2-micro ParameterKey=KeyName,ParameterValue=jan8 ParameterKey=VpcId,ParameterValue=vpc-09553104ec99decb8 ParameterKey=Subnets,ParameterValue=subnet-0e3b397242dff1d65  
                    else
                      aws cloudformation create-stack  --stack-name jenkinsHello --template-body file://2-ec2.yaml  --parameters ParameterKey=InstanceType,ParameterValue=t2-micro ParameterKey=KeyName,ParameterValue=jan8 ParameterKey=VpcId,ParameterValue=vpc-09553104ec99decb8 ParameterKey=Subnets,ParameterValue=subnet-0e3b397242dff1d65
                    fi
                    '''

                    sh '''#!/usr/bin/env bash
                    sleep 30
                    export status=$(aws cloudformation describe-stacks  --stack-name=jenkinsHello --query Stacks[].StackStatus --output text)
                    while [[ $status != *COMPLETE &&  $status != *FAILED ]]; do
                    sleep 30
                    export status=$(aws cloudformation describe-stacks  --stack-name=jenkinsHello --query Stacks[].StackStatus --output text)
                    done
                    echo $status
                    [[ $status == "UPDATE_COMPLETE" || $status == "CREATE_COMPLETE" ]]
                    '''
                } // withCredentials
        } // steps        
    } //stage
  } //stages
} // pipeline
