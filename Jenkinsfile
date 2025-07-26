pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID      = credentials('account_id')
        AWS_DEFAULT_REGION  = 'us-east-1'
        AWS_ACCESS_KEY_ID   = credentials('aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
    }

    stages {
         stage('Infrastructure Deployment') {
            agent {
                docker {
                    image 'hashicorp/terraform:light'
                }
            }
        stage('Infrastructure Deployment') {
            steps {
                script {
                    sh 'terraform init'
                    sh 'terraform validate'
                    sh 'terraform plan'
                    sh "terraform ${action} --auto-approve"
                }
            }
        }
    }
}
}
