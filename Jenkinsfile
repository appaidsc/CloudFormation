pipeline {
    agent any

    parameters {
        choice(name: 'DEPLOY_TYPE', choices: ['s3', 'ec2'], description: 'Choose the resource type to deploy')

        choice(name: 'AWS_REGION', choices: ['us-east-1', 'us-west-1', 'ap-south-1', 'eu-central-1'], description: 'Choose AWS region')

        string(name: 'STACK_NAME', defaultValue: 'example-stack', description: 'CloudFormation Stack Name')

        choice(name: 'TEMPLATE_FILE', choices: ['01_s3cft.yml', '02_ec2cft.yml'], description: 'Select appropriate CloudFormation template')

        string(name: 'BUCKET_NAME', defaultValue: '', description: 'Bucket name (Only for S3)')

        string(name: 'INSTANCE_NAME', defaultValue: '', description: 'EC2 instance name (Only for EC2)')
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
                    def baseCmd = """
                        aws cloudformation deploy \
                          --stack-name ${params.STACK_NAME} \
                          --template-file ${params.TEMPLATE_FILE} \
                          --region ${params.AWS_REGION}
                    """

                    if (params.DEPLOY_TYPE == 's3') {
                        if (!params.BUCKET_NAME?.trim()) {
                            error "BUCKET_NAME must be provided for S3 deployments."
                        }
                        baseCmd += " --parameter-overrides BucketName=${params.BUCKET_NAME}"
                    } else if (params.DEPLOY_TYPE == 'ec2') {
                        if (!params.INSTANCE_NAME?.trim()) {
                            error "INSTANCE_NAME must be provided for EC2 deployments."
                        }
                        baseCmd += " --parameter-overrides InstanceName=${params.INSTANCE_NAME}"
                    } else {
                        error "Invalid DEPLOY_TYPE selected."
                    }

                    echo "Running CloudFormation deployment..."
                    sh baseCmd
                }
            }
        }

        stage('Show EC2 Outputs') {
            when {
                expression { params.DEPLOY_TYPE == 'ec2' }
            }
            steps {
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
