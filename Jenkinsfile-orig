pipeline {
    agent any
    stages {
        stage('Fetch Parameters') {
            steps {
                script {
                    // Fetching all parameters including stack name and region
                    env.REGION = sh(script: "aws ssm get-parameter --name 'Region' --with-decryption --query 'Parameter.Value' --output text", returnStdout: true).trim()
                    env.STACK_NAME = sh(script: "aws ssm get-parameter --name 'StackName' --with-decryption --query 'Parameter.Value' --output text", returnStdout: true).trim()
                    env.KEY_NAME = sh(script: "aws ssm get-parameter --name 'KeyName' --with-decryption --query 'Parameter.Value' --output text", returnStdout: true).trim()
                    env.INSTANCE_TYPE = sh(script: "aws ssm get-parameter --name 'InstanceType' --with-decryption --query 'Parameter.Value' --output text", returnStdout: true).trim()
                    env.IMAGE_ID = sh(script: "aws ssm get-parameter --name 'ImageId' --with-decryption --query 'Parameter.Value' --output text", returnStdout: true).trim()
                    env.SECURITY_GROUP_ID = sh(script: "aws ssm get-parameter --name 'SecurityGroupId' --with-decryption --query 'Parameter.Value' --output text", returnStdout: true).trim()
                    env.SUBNET_ID = sh(script: "aws ssm get-parameter --name 'SubnetId' --with-decryption --query 'Parameter.Value' --output text", returnStdout: true).trim()
                }
            }
        }
        stage('Deploy Master Stack') {
            steps {
                script {
                    // Build the full path to the master template file
                    def templatePath = "${WORKSPACE}/master.yaml"
                    // Check if the stack exists
                    def checkStack = sh(script: "aws cloudformation describe-stacks --stack-name ${env.STACK_NAME} --region ${env.REGION}", returnStatus: true)
                    if (checkStack == 0) {
                        // Update the stack if it exists
                        echo "Updating existing stack..."
                        sh "aws cloudformation update-stack --stack-name ${env.STACK_NAME} --template-body file://${templatePath} --parameters \
                            ParameterKey=KeyName,ParameterValue=${KEY_NAME} \
                            ParameterKey=InstanceType,ParameterValue=${INSTANCE_TYPE} \
                            ParameterKey=ImageId,ParameterValue=${IMAGE_ID} \
                            ParameterKey=SecurityGroupId,ParameterValue=${SECURITY_GROUP_ID} \
                            ParameterKey=SubnetId,ParameterValue=${SUBNET_ID} \
                            --capabilities CAPABILITY_NAMED_IAM --region ${env.REGION}"
                    } else {
                        // Create the stack if it does not exist
                        echo "Creating new stack..."
                        sh "aws cloudformation create-stack --stack-name ${env.STACK_NAME} --template-body file://${templatePath} --parameters \
                            ParameterKey=KeyName,ParameterValue=${KEY_NAME} \
                            ParameterKey=InstanceType,ParameterValue=${INSTANCE_TYPE} \
                            ParameterKey=ImageId,ParameterValue=${IMAGE_ID} \
                            ParameterKey=SecurityGroupId,ParameterValue=${SECURITY_GROUP_ID} \
                            ParameterKey=SubnetId,ParameterValue=${SUBNET_ID} \
                            --capabilities CAPABILITY_NAMED_IAM --region ${env.REGION}"
                    }
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
        always {
            echo 'Deployment process completed.'
        }
    }
}
