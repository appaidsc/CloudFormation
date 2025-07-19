pipeline {
    agent any

    parameters {
        choice(name: 'ResourceType', choices: ['S3', 'EC2'], description: 'Select the type of resource to create')
        choice(name: 'Region', choices: ['us-east-1', 'us-west-1', 'us-west-2'], description: 'Select AWS Region')
        string(name: 'StackName', defaultValue: 'example-stack', description: 'CloudFormation Stack Name')
        choice(name: 'TemplateFile', choices: ['01_s3cft.yml', '02_ec2cft.yml'], description: 'Select Template File')
        string(name: 'BucketName', defaultValue: '', description: 'Bucket Name (Only if S3 selected)')
        string(name: 'InstanceName', defaultValue: '', description: 'EC2 Instance Name (Only if EC2 selected)')
    }

    environment {
        AWS_DEFAULT_REGION = "${params.Region}"
    }

    stages {
        stage('Checkout Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/riteshbehal/CloudFormation.git'
            }
        }

        stage('Deploy CloudFormation Stack') {
            steps {
                script {
                    echo "Running CloudFormation deployment..."

                    def parameterOverrides = ''
                    if (params.ResourceType == 'S3') {
                        if (!params.BucketName) {
                            error("BucketName is required for S3 deployment.")
                        }
                        parameterOverrides = "BucketName=${params.BucketName}"
                    } else if (params.ResourceType == 'EC2') {
                        if (!params.InstanceName) {
                            error("InstanceName is required for EC2 deployment.")
                        }
                        parameterOverrides = "InstanceName=${params.InstanceName}"
                    }

                    sh """
                        aws cloudformation deploy \
                          --stack-name ${params.StackName} \
                          --template-file ${params.TemplateFile} \
                          --region ${params.Region} \
                          --parameter-overrides ${parameterOverrides}
                    """
                }
            }
        }

        stage('Show EC2 Outputs') {
            when {
                expression { return params.ResourceType == 'EC2' }
            }
            steps {
                script {
                    echo "Fetching EC2 Outputs..."
                    sh """
                        aws cloudformation describe-stacks \
                          --stack-name ${params.StackName} \
                          --region ${params.Region} \
                          --query "Stacks[0].Outputs"
                    """
                }
            }
        }
    }
}
