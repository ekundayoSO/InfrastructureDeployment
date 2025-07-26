pipeline {
    agent any
    
    parameters {
        choice(
            name: 'ACTION',
            choices: ['plan', 'apply', 'destroy'],
            description: 'Terraform action to perform'
        )
        string(
            name: 'WORKSPACE',
            defaultValue: 'default',
            description: 'Terraform workspace'
        )
    }
    
    environment {
        AWS_ACCOUNT_ID      = credentials('account_id')
        AWS_DEFAULT_REGION  = 'us-east-1'
        AWS_ACCESS_KEY_ID   = credentials('aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
        TF_IN_AUTOMATION    = 'true'
        TF_INPUT            = 'false'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Terraform Init') {
            agent {
                docker {
                    image 'hashicorp/terraform:light'
                    args '--entrypoint=""'
                }
            }
            steps {
                sh 'terraform init'
                sh "terraform workspace select ${params.WORKSPACE} || terraform workspace new ${params.WORKSPACE}"
            }
        }
        
        stage('Terraform Validate') {
            agent {
                docker {
                    image 'hashicorp/terraform:light'
                    args '--entrypoint=""'
                }
            }
            steps {
                sh 'terraform validate'
            }
        }
        
        stage('Terraform Plan') {
            agent {
                docker {
                    image 'hashicorp/terraform:light'
                    args '--entrypoint=""'
                }
            }
            steps {
                sh 'terraform plan -out=tfplan'
                archiveArtifacts artifacts: 'tfplan', fingerprint: true
            }
        }
        
        stage('Terraform Apply/Destroy') {
            when {
                anyOf {
                    expression { params.ACTION == 'apply' }
                    expression { params.ACTION == 'destroy' }
                }
            }
            agent {
                docker {
                    image 'hashicorp/terraform:light'
                    args '--entrypoint=""'
                }
            }
            steps {
                script {
                    if (params.ACTION == 'apply') {
                        sh 'terraform apply tfplan'
                    } else if (params.ACTION == 'destroy') {
                        sh 'terraform destroy --auto-approve'
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        failure {
            emailext (
                subject: "Pipeline Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                body: "The pipeline ${env.JOB_NAME} failed at build ${env.BUILD_NUMBER}. Please check the logs.",
                to: "${env.CHANGE_AUTHOR_EMAIL}"
            )
        }
    }
}
