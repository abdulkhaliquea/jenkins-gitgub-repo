pipeline {
    agent any

    tools {
        terraform 'terraform-latest'

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                sh '''
                terraform init \
                  -backend-config="bucket=myuniquebucketnew" \
                  -backend-config="key=infrastructure/terraform.tfstate" \
                  -backend-config="region=us-east-1"
                '''
            }
        }

        stage('Terraform Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
            }
        }

        stage('Approval Gate') {
            steps {
                // Wrap the input step in a timeout so it doesn't consume an executor indefinitely
                timeout(time: 1, unit: 'HOURS') {
                    script {
                        // This pauses the pipeline and prompts the user
                        input message: 'Review the terraform plan. Do you want to apply these changes?', 
                              ok: 'Approve and Apply'
                    }
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                // Apply the exact plan that was reviewed
                sh 'terraform apply -auto-approve tfplan'
            }
        }
    }
}
}