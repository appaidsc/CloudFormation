pipeline {
    agent any

    parameters {
        choice(name: 'DEPLOY_TYPE', choices: ['s3', 'ec2'], description: 'Choose resource type to deploy')

        string(name: 'STACK_NAME', defaultValue: 'example-stack', description: 'CloudFormation Stack Name')

        // Visible only if S3 is selected
        string(name: 'BUCKET_NAME', defaultValue: '', description: 'S3 Bucket Name (Only for S3)')

        // Visible only if EC2 is selected
        string(name: 'INSTANCE_NAME', defaultValue: '', description: 'EC2 Instance Name (Only for EC2)')

        // Template file selection
        choice(name: 'TEMPLATE_FILE', choices: ['01_s3cft.yml', '02_ec2cft.yml'], description: 'Choose CloudFormation Template File')
    }

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
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
                    def deployCmd = """
                        aws cloudformation deploy \
                          --stack-name ${params.STACK_NAME} \
                          --template-file ${params.TEMPLATE_FILE} \
                          --region ${AWS_DEFAULT_REGION}
                    """

                    if (params.DEPLOY_TYPE == "s3") {
                        deployCmd += " --parameter-overrides BucketName=${params.BUCKET_NAME}"
                    } else if (params.DEPLOY_TYPE == "ec2") {
                        deployCmd += " --parameter-overrides InstanceName=${params.INSTANCE_NAME}"
                    }

                    echo "Running deployment command..."
                    sh deployCmd
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
                      --region ${AWS_DEFAULT_REGION} \
                      --query "Stacks[0].Outputs" \
                      --output table
                """
            }
        }
    }
}
