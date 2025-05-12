pipeline {
    agent any

    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

    stages {
        stage('Checkout') {
            steps {
                // Clone the correct repository
                git url: 'https://github.com/theewizardone/Terraform--Jenkins.git', branch: 'main'
            }
        }

        stage('Terraform Init & Plan') {
            steps {
                dir('terraform') {
                    bat '''
                        terraform init
                        terraform plan -out=tfplan
                        terraform show -no-color tfplan > tfplan.txt
                    '''
                }
            }
        }

        stage('Approval') {
            when {
                not {
                    equals expected: true, actual: params.autoApprove
                }
            }
            steps {
                script {
                    def planText = readFile('terraform/tfplan.txt')
                    input(
                        message: 'Do you want to apply this plan?',
                        parameters: [
                            text(name: 'Plan', description: 'Please review the plan below:', defaultValue: planText)
                        ]
                    )
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                dir('terraform') {
                    bat 'terraform apply -input=false tfplan'
                }
            }
        }
    }

    post {
        success {
            echo '✅ Terraform pipeline completed successfully.'
        }
        failure {
            echo '❌ Pipeline failed. Check logs for details.'
        }
    }
}
