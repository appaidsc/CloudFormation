pipeline {
    agent any

    parameters {
        choice(name: 'DEPLOY_TYPE', choices: ['s3', 'ec2'], description: 'Choose resource type to deploy')

        choice(name: 'AWS_REGION', choices: ['us-east-1', 'us-west-1', 'ap-south-1'], description: 'Select AWS Region')

        choice(name: 'TEMPLATE_FILE', choices: ['01_s3cft.yml', '02_ec2cft.yml'], description: 'Choose the template file to deploy')

        string(name: 'STACK_NAME', defaultValue: 'example-stack', description: 'CloudFormation Stack Name')

        string(name: 'BUCKET_NAME', defaultValue: '', description: 'Enter Bucket Name (Required for S3)')

        string(name: 'INSTANCE_NAME', defaultValue: '', description: 'Enter Instance Name (Required for EC2)')
    }

    environment {
        AWS_DEFAULT_REGION = "${params.AWS_REGION}"
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

                    def deployCommand = """
                        aws cloudformation deploy \
                          --stack-name ${params.STACK_NAME} \
                          --template-file ${params.TEMPLATE_FILE} \
                          --region ${params.AWS_REGION}
                    """

                    // Add parameter overrides based on deployment type
                    if (params.DEPLOY_TYPE == 's3') {
                        if (!params.BUCKET_NAME?.trim()) {
                            error "BUCKET_NAME is required for S3 deployment"
                        }
                        deployCommand += " --parameter-overrides BucketName=${params.BUCKET_NAME}"
                    } else if (params.DEPLOY_TYPE == 'ec2') {
                        if (!params.INSTANCE_NAME?.trim()) {
                            error "INSTANCE_NAME is required for EC2 deployment"
                        }
                        deployCommand += " --parameter-overrides InstanceName=${params.INSTANCE_NAME}"
                    }

                    sh deployCommand
                }
            }
        }

        stage('Show EC2 Outputs') {
            when {
                expression { return params.DEPLOY_TYPE == 'ec2' }
            }
            steps {
                echo "Fetching EC2 Outputs..."
                sh """
                    aws cloudformation describe-stacks \
                      --stack-name ${params.STACK_NAME} \
                      --region ${params.AWS_REGION} \
                      --query "Stacks[0].Outputs" \
                      --output table
                """
            }
        }
    }
}
