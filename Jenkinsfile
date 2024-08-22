pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1' // Set your default AWS region
        S3_BUCKET = 'rahul-bucket-v2' // S3 bucket to store packaged templates
        STACK_NAME_1 = 'DynamoDBGlobalTableStack' // CloudFormation stack name for DynamoDB
        STACK_NAME_2 = 'IAMRoleStack' // CloudFormation stack name for IAM Role
        STACK_NAME_3 = 'LambdaAndEventMappingStack' // CloudFormation stack name for Lambda and Event Source Mapping
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install SAM CLI') {
            steps {
                script {
                    // Install AWS SAM CLI if not installed
                    sh 'if ! command -v sam &> /dev/null; then pip install aws-sam-cli; fi'
                }
            }
        }

        stage('Deploy Stack 1: DynamoDB Global Table') {
            steps {
                script {
                    // Validate the SAM template for Stack 1
                    // sh 'sam validate --template-file DynamoDBStack.yaml --region ${AWS_REGION}'

                    // Package the SAM template for Stack 1
                    sh "sam package --template-file DynamoDBStack.yaml --s3-bucket ${S3_BUCKET} --output-template-file packaged-stack1.yaml --region ${AWS_REGION}"

                    // Deploy the SAM template for Stack 1
                    sh "sam deploy --template-file packaged-stack1.yaml --stack-name ${STACK_NAME_1} --capabilities CAPABILITY_IAM --region ${AWS_REGION}"

                    // Capture the outputs needed for subsequent stacks
                    sh 'aws cloudformation describe-stacks --stack-name ${STACK_NAME_1} --region ${AWS_REGION} --query "Stacks[0].Outputs" > stack1-outputs.json'
                }
            }
        }

        stage('Deploy Stack 2: IAM Role') {
            steps {
                script {
                    // Validate the SAM template for Stack 2
                    // sh 'sam validate --template-file IAMroleStack.yaml --region ${AWS_REGION}'

                    // Package the SAM template for Stack 2
                    sh "sam package --template-file IAMroleStack.yaml --s3-bucket ${S3_BUCKET} --output-template-file packaged-stack2.yaml --region ${AWS_REGION}"

                    // Deploy the SAM template for Stack 2
                    sh "sam deploy --template-file packaged-stack2.yaml --stack-name ${STACK_NAME_2} --capabilities CAPABILITY_IAM --region ${AWS_REGION}"

                    // Capture the outputs needed for subsequent stacks
                    sh 'aws cloudformation describe-stacks --stack-name ${STACK_NAME_2} --region ${AWS_REGION} --query "Stacks[0].Outputs" > stack2-outputs.json'
                }
            }
        }

        stage('Deploy Stack 3: Lambda and Event Source Mapping') {
            steps {
                script {
                    // Extract necessary ARNs from previous stack outputs
                    def globalTableStreamArn = sh(script: "jq -r '.[] | select(.OutputKey==\"GlobalTableStreamArn\") | .OutputValue' stack1-outputs.json", returnStdout: true).trim()
                    def lambdaRoleArn = sh(script: "jq -r '.[] | select(.OutputKey==\"LambdaRoleArn\") | .OutputValue' stack2-outputs.json", returnStdout: true).trim()

                    // Validate the SAM template for Stack 3
                    // sh 'sam validate --template-file LambdaStack.yaml --region ${AWS_REGION}'

                    // Package the SAM template for Stack 3
                    sh "sam package --template-file LambdaStack.yaml --s3-bucket ${S3_BUCKET} --output-template-file packaged-stack3.yaml --region ${AWS_REGION}"

                    // Deploy the SAM template for Stack 3
                    sh """
                        sam deploy --template-file packaged-stack3.yaml --stack-name ${STACK_NAME_3} \
                        --capabilities CAPABILITY_IAM --region ${AWS_REGION} \
                        --parameter-overrides LambdaRoleArn=${lambdaRoleArn} GlobalTableStreamArn=${globalTableStreamArn}
                    """
                }
            }
        }

        stage('Clean Up') {
            steps {
                cleanWs()
            }
        }
    }

    post {
        always {
            echo 'Deployment finished.'
        }
    }
}
