pipeline {
    agent any

    parameters {
        choice(name: 'TERRAFORM_ACTION', choices: ['apply', 'destroy'], description: 'Choose Terraform action to run')
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
        AWS_DEFAULT_REGION    = 'us-east-1'
        AWS_KEYPAIR_NAME      = credentials('aws_keypair_name')
    }

    stages {
        stage('Git Checkout') {
            steps {
                git 'https://github.com/ekundayoSO/InfrastructureDeployment.git'
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
                sh 'terraform validate'
                sh 'terraform plan -var="key_name=${AWS_KEYPAIR_NAME}"'
            }
        }

        stage('Terraform ${params.TERRAFORM_ACTION}') {
            steps {
                script {
                    if (params.TERRAFORM_ACTION == 'apply' || params.TERRAFORM_ACTION == 'destroy') {
                        sh "terraform ${params.TERRAFORM_ACTION} -auto-approve -var=\"key_name=${AWS_KEYPAIR_NAME}\""
                    } else {
                        error "Invalid Terraform action: ${params.TERRAFORM_ACTION}"
                    }
                }
            }
        }
    }
}
