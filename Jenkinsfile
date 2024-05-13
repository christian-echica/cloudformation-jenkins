pipeline {
    agent any
    environment {
        STACK_NAME = "ecinstance10-master"
        MASTER_TEMPLATE = "master.yaml" 
        REGION = "ap-south-1"
    }
    stages {
        stage('Checkout Repository') {
            steps {
                checkout scm
            }
        }
        stage('Deploy Master Stack') {
            steps {
                script {
                    // Build the full path to the master template file
                    def templatePath = "${WORKSPACE}/${MASTER_TEMPLATE}"
                    // Check if the stack exists
                    def checkStack = sh(script: "aws cloudformation describe-stacks --stack-name ${env.STACK_NAME} --region ${env.REGION}", returnStatus: true)
                    if (checkStack == 0) {
                        // Update the stack if it exists
                        echo "Updating existing stack..."
                        sh "aws cloudformation update-stack --stack-name ${env.STACK_NAME} --template-body file://${templatePath} --capabilities CAPABILITY_NAMED_IAM --region ${env.REGION}"
                    } else {
                        // Create the stack if it does not exist
                        echo "Creating new stack..."
                        sh "aws cloudformation create-stack --stack-name ${env.STACK_NAME} --template-body file://${templatePath} --capabilities CAPABILITY_NAMED_IAM --region ${env.REGION}"
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
