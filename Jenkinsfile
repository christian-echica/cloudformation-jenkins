pipeline {
    agent any
    environment {
        // Define AWS_DEFAULT_REGION to ensure all AWS CLI commands use this region
        AWS_DEFAULT_REGION = 'ap-south-1' // Set your AWS region here
        S3_BUCKET_URI = 's3://cft-templates-christianechica/cft-jenkins-demo/' // S3 bucket URI
    }
    stages {
        stage('Upload CloudFormation Templates to S3') {
            steps {
                script {
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding', 
                        credentialsId: 'aws-creds', 
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID', 
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    ]]) {
                        // Upload all .yml files in the workspace to S3
                        sh "aws s3 cp ${WORKSPACE}/ ${S3_BUCKET_URI} --recursive --exclude '*' --include '*.yml'"
                    } // Closing the withCredentials block
                }
            }
        }
        stage('Verify Environment') {
            steps {
                script {
                    sh "ls -la ${WORKSPACE}/"  // List all files in the workspace directory to check file presence
                    sh "cat ${WORKSPACE}/master.yml"  // Optionally, output the content of the master.yml to verify its correctness
                }
            }
        }
        stage('Fetch Parameters') {
            steps {
                script {
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding', 
                        credentialsId: 'aws-creds', 
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID', 
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    ]]) {
                        // Fetching all parameters including stack name and region
                        env.AWS_REGION = sh(script: "aws ssm get-parameter --name 'Region' --with-decryption --query 'Parameter.Value' --output text", returnStdout: true).trim()
                        env.STACK_NAME = sh(script: "aws ssm get-parameter --name 'StackName' --with-decryption --query 'Parameter.Value' --output text", returnStdout: true).trim()
                        env.KEY_NAME = sh(script: "aws ssm get-parameter --name 'KeyName' --with-decryption --query 'Parameter.Value' --output text", returnStdout: true).trim()
                        env.INSTANCE_TYPE = sh(script: "aws ssm get-parameter --name 'InstanceType' --with-decryption --query 'Parameter.Value' --output text", returnStdout: true).trim()
                        env.IMAGE_ID = sh(script: "aws ssm get-parameter --name 'ImageId' --with-decryption --query 'Parameter.Value' --output text", returnStdout: true).trim()
                        env.SECURITY_GROUP_ID = sh(script: "aws ssm get-parameter --name 'SecurityGroupId' --with-decryption --query 'Parameter.Value' --output text", returnStdout: true).trim()
                        env.SUBNET_ID = sh(script: "aws ssm get-parameter --name 'SubnetId' --with-decryption --query 'Parameter.Value' --output text", returnStdout: true).trim()
                    } // Closing the withCredentials block
                }
            }
        }
        stage('Deploy Master Stack') {
            steps {
                script {
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding', 
                        credentialsId: 'aws-creds', 
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID', 
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    ]]) {
                        // Build the full path to the master template file
                        def templatePath = "${WORKSPACE}/master.yml"
                        // Check if the stack exists
                        def describeOutput = sh(script: "aws cloudformation describe-stacks --stack-name ${env.STACK_NAME} --region ${env.REGION}", returnStdout: true, returnStatus: true)
                        if (describeOutput == 0) {
                            // Update the stack if it exists
                            echo "Updating existing stack..."
                            sh """
                            aws cloudformation update-stack --stack-name ${env.STACK_NAME} \
                            --template-body file://\${WORKSPACE}/master.yml \
                            --parameters \
                                ParameterKey=KeyName,ParameterValue=\${env.KEY_NAME} \
                                ParameterKey=InstanceType,ParameterValue=\${env.INSTANCE_TYPE} \
                                ParameterKey=ImageId,ParameterValue=\${env.IMAGE_ID} \
                                ParameterKey=SecurityGroupId,ParameterValue=\${env.SECURITY_GROUP_ID} \
                                ParameterKey=SubnetId,ParameterValue=\${env.SUBNET_ID} \
                            --capabilities CAPABILITY_NAMED_IAM \
                            --region \${env.REGION}
                            """                        } else {
                            // Create the stack if it does not exist
                            echo "Creating new stack..."
                            sh "aws cloudformation create-stack --stack-name DEV-INFRASTRUCTURE --template-body file://${templatePath} --parameters \
                            ParameterKey=KeyName,ParameterValue=${env.KEY_NAME} \
                            ParameterKey=InstanceType,ParameterValue=${env.INSTANCE_TYPE} \
                            ParameterKey=ImageId,ParameterValue=${env.IMAGE_ID} \
                            ParameterKey=SecurityGroupId,ParameterValue=${env.SECURITY_GROUP_ID} \
                            ParameterKey=SubnetId,ParameterValue=${env.SUBNET_ID} \
                            --capabilities CAPABILITY_NAMED_IAM --region ${env.AWS_REGION}"                        }
                    } // Closing the withCredentials block
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
