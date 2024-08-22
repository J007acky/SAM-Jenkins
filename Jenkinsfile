pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1' // Set your default AWS region
        S3_BUCKET = 'rahul-bucket-v2' // S3 bucket to store packaged templates
        STACK_NAME_1 = 'DynamoDBstack' // CloudFormation stack name for DynamoDB
        STACK_NAME_2 = 'LambdaRole' // CloudFormation stack name for IAM Role
        STACK_NAME_3 = 'LambdaStack' // CloudFormation stack name for Lambda and Event Source Mapping
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
                    withAWS(credentials: 'aws-access', region: "$AWS_REGION") {
                        try {
                            // Package the SAM template for Stack 1
                            sh "sam package --template-file ka-me-ha-me-ha-archives.yaml --s3-bucket ${S3_BUCKET} --output-template-file output1.yaml --region ${AWS_REGION}"

                            // Deploy the SAM template for Stack 1
                            sh "sam deploy --template-file output1.yaml --stack-name ${STACK_NAME_1} --capabilities CAPABILITY_IAM --region ${AWS_REGION}"

                            // Capture the GlobalTableStreamArn output
                            sh "aws cloudformation describe-stacks --stack-name ${STACK_NAME_1} --region ${AWS_REGION} --query 'Stacks[0].Outputs[?OutputKey==`GlobalTableStreamArn`].OutputValue' --output text > globalTableStreamArn.txt"
                        } catch (Exception e) {
                            sh 'echo "No changes to deploy for stack ${STACK_NAME_1}. Continuing..."'
                        }
                    }
                }
            }
        }

        stage('Deploy Stack 2: IAM Role') {
            steps {
                script {
                    withAWS(credentials: 'aws-access', region: "$AWS_REGION") {
                        try {
                            // Package the SAM template for Stack 2
                            sh "sam package --template-file lambda-role.yaml --s3-bucket ${S3_BUCKET} --output-template-file output2.yaml --region ${AWS_REGION}"

                            // Deploy the SAM template for Stack 2
                            sh "sam deploy --template-file output2.yaml --stack-name ${STACK_NAME_2} --capabilities CAPABILITY_NAMED_IAM --region ${AWS_REGION}"

                            // Capture the LambdaRoleArn output
                            sh "aws cloudformation describe-stacks --stack-name ${STACK_NAME_2} --region ${AWS_REGION} --query 'Stacks[0].Outputs[?OutputKey==`LambdaRoleArn`].OutputValue' --output text > lambdaRoleArn.txt"
                        } catch (Exception e) {
                            sh 'echo "No changes to deploy for stack ${STACK_NAME_2}. Continuing..."'
                        }
                    }
                }
            }
        }

        stage('Deploy Stack 3: Lambda and Event Source Mapping') {
            steps {
                script {
                    withAWS(credentials: 'aws-access', region: "$AWS_REGION") {
                        try {
                            // Read the captured outputs
                            def globalTableStreamArn = readFile('globalTableStreamArn.txt').trim()
                            def lambdaRoleArn = readFile('lambdaRoleArn.txt').trim()

                            
                            // Package the SAM template for Stack 3
                            sh "sam package --template-file ka-me-ha-me-ha-enabler.yaml --s3-bucket ${S3_BUCKET} --output-template-file output3.yaml --region ${AWS_REGION}"

                            // Deploy the SAM template for Stack 3
                            sh "sam deploy --template-file output3.yaml --stack-name ${STACK_NAME_3} --capabilities CAPABILITY_IAM --region ${AWS_REGION} --parameter-overrides ParameterKey=GlobalTableStreamArn,ParameterValue=${GLOBAL_TABLE_STREAM_ARN} ParameterKey=LambdaRoleArn,ParameterValue=${LAMBDA_ROLE_ARN}"
                        } catch (Exception e) {
                            sh 'echo "No changes to deploy for stack ${STACK_NAME_3}. Continuing..."'
                            throw e
                        }
                    }
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
