pipeline {
    agent any
    environment{
        AWS_REGION = 'ap-south-1'
        S3_BUCKET = 'rahul-bucket-v2'
        STACK_DB = 'DynamoDBStack'
        STACK_LAMBDA = 'LambdaStack'
        STACK_ROLE = 'RoleStack'
    }
    stages{
        stage('Checkout'){
            steps{
                checkout scm
            }
        }
        stage('Verify AWS/SAM CLI'){
            steps{
                script{
                    sh 'if ! command -v sam &> /dev/null; then pip install aws-sam-cli; fi'
                }
            }
        }
        stage('Deploying DynamoDB Stack'){
            steps{
                script{
                    withAWS(credentials: 'aws-access', region: "$AWS_REGION"){
                    try {
                        // Packaging SAM templates
                        sh "sam package --template-file ka-me-ha-me-ha-archives.yaml --s3-bucket ${S3_BUCKET} --output-template-file DynamoStack.yaml --region ${AWS_REGION}"
                        // Deploying the Packaged templates
                        sh "sam deploy --template-file DynamoStack.yaml --stack-name ${STACK_NAME_1} --capabilities CAPABILITY_IAM --region ${AWS_REGION}"
                    }
                    catch (Exception e){
                        sh 'echo "No changes to deploy for stack ${STACK_NAME_1}. Continuing..."'
                    }
                
                }
                }
                }
            }
        stage('Deploying Lambda Role Stack'){
            steps{
                script{
                     withAWS(credentials: 'aws-access', region: "$AWS_REGION"){
                    try {
                        // Packaging SAM templates
                        sh "sam package --template-file lambda-role.yaml --s3-bucket ${S3_BUCKET} --output-template-file Roles.yaml --region ${AWS_REGION}"
                        // Deploying the Packaged templates
                        sh "sam deploy --template-file Roles.yaml --stack-name ${STACK_NAME_2} --capabilities CAPABILITY_NAMED_IAM --region ${AWS_REGION}"
                    }
                    catch (Exception e){
                        sh 'echo "No changes to deploy for stack ${STACK_NAME_2}. Continuing..."'
                    }
                
                }
                }
                }
            }
        stage('Deploying Lambda Stack'){
            steps{
                script{
                    withAWS(credentials: 'aws-access', region: "$AWS_REGION"){
                    try {
                        // Packaging SAM templates
                        sh "sam package --template-file ka-me-ha-me-ha-enabler.yaml --s3-bucket ${S3_BUCKET} --output-template-file Lambda.yaml --region ${AWS_REGION}"
                        // Deploying the Packaged templates
                        sh "sam deploy --template-file Lambda.yaml --stack-name ${STACK_NAME_1} --capabilities CAPABILITY_IAM --region ${AWS_REGION}"
                    }
                    catch (Exception e){
                        sh 'echo "No changes to deploy for stack ${STACK_NAME_3}. Continuing..."'
                    }
                
                }
                }
                }
            }
        stage('WorkSpace Cleanup'){
            steps{
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