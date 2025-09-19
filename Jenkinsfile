properties([
    parameters([
        string(
            defaultValue: 'dev',
            name: 'Environment',
            description: 'dev (e.g., dev, staging, prod)'
        ),
        choice(
            choices: ['plan', 'apply', 'destroy'], 
            name: 'Terraform_Action',
            description: 'Select the Terraform action to perform'
        )
    ])
])

pipeline {
    agent any

    stages {
        stage('Preparing') {
            steps {
                sh 'echo Preparing'
            }
        }

        stage('Git Pulling') {
            steps {
                git branch: 'main', url: 'https://github.com/monallonare/INFRA.git'
            }
        }

        stage('Init') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                sh 'terraform -chdir=eks/ init'
                }
            }
        }

        stage('Validate') {
            steps {
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                    sh 'terraform -chdir=eks/ validate'
                }
            }
        }

        stage('Action') {
            steps {
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                    script {
                        def tfvars = "${params.Environment}.tfvars"

                        if (params.Terraform_Action == 'plan') {
                            sh "terraform -chdir=eks/ plan -var-file=${tfvars}"
                        } else if (params.Terraform_Action == 'apply') {
                            sh "terraform -chdir=eks/ apply -var-file=${tfvars} -auto-approve"
                        } else if (params.Terraform_Action == 'destroy') {
                            sh "terraform -chdir=eks/ destroy -var-file=${tfvars} -auto-approve"
                        } else {
                            error "Invalid value for Terraform_Action: ${params.Terraform_Action}"
                        }
                    }
                }
            }
        }
    }
}
