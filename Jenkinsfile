pipeline {
    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
    } 
    
    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

    agent any

    stages {
        stage('Checkout') {
            steps {
                script {
                    deleteDir()  // Clears previous workspace before cloning
                    retry(3) {
                        checkout([$class: 'GitSCM',
                            branches: [[name: '*/main']],
                            userRemoteConfigs: [[
                                url: 'https://github.com/ashish-2106/Terraform-Jenkins.git',
                                credentialsId: 'your-credential-id'
                            ]],
                            extensions: [[$class: 'CloneOption', timeout: 30, noTags: false]]
                        ])
                    }
                }
            }
        }

        stage('Verify Files') {
            steps {
                script {
                    sh '''
                    echo "Checking repository structure..."
                    pwd
                    ls -la
                    '''
                }
            }
        }

        stage('Terraform Init') {
            steps {
                dir('Terraform-Jenkins') {  // Ensure correct directory
                    sh 'terraform init'
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                dir('Terraform-Jenkins') {
                    sh '''
                    terraform plan -out=tfplan
                    terraform show -no-color tfplan > tfplan.txt
                    '''
                }
            }
        }

        stage('Approval') {
           when {
               not { equals expected: true, actual: params.autoApprove }
           }
           steps {
               script {
                   def plan = readFile 'Terraform-Jenkins/tfplan.txt'
                   input message: "Do you want to apply the plan?",
                   parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
               }
           }
       }

        stage('Terraform Apply') {
            steps {
                dir('Terraform-Jenkins') {
                    sh 'terraform apply -auto-approve tfplan'
                }
            }
        }
    }

    post {
        success {
            echo "Terraform execution completed successfully!"
        }
        failure {
            echo "Terraform execution failed!"
        }
    }
}
