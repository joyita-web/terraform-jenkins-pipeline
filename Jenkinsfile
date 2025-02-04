pipeline {
    agent any

    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
        choice(name: 'action', choices: ['apply', 'destroy'], description: 'Select the action to perform')
        string(name: 'AWS_REGION', defaultValue: 'ap-south-1', description: 'Enter AWS Region')
        string(name: 'AMI', defaultValue: 'ami-0f5ee92e2d63afc18', description: 'Enter AMI ID')
        choice(name: 'INSTANCE_TYPE', choices: ['t2.micro', 't2.small', 't2.medium'], description: 'Select Instance Type')
        string(name: 'NAME_TAG', description: 'Enter Name Tag for EC2 Instance')
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        AWS_DEFAULT_REGION    = params.AWS_REGION  // ✅ User input
        AMI                  = params.AMI         // ✅ User input
        INSTANCE_TYPE        = params.INSTANCE_TYPE  // ✅ User input
        NAME_TAG             = params.NAME_TAG   // ✅ User input
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/joyita-web/terraform-jenkins-pipeline.git'
            }
        }
        stage('Terraform init') {
            steps {
                sh 'terraform init'
            }
        }
        stage('Plan') {
            steps {
                sh 'terraform plan -var="ami=${AMI}" -var="instance_type=${INSTANCE_TYPE}" -var="name_tag=${NAME_TAG}" -out tfplan'
                sh 'terraform show -no-color tfplan > tfplan.txt'  // Ensure plan output is saved
            }
        }  // <-- This closing bracket was missing

        stage('Apply / Destroy') {
            steps {
                script {
                    if (params.action == 'apply') {
                        if (!params.autoApprove) {
                            def plan = readFile 'tfplan.txt'
                            input message: "Do you want to apply the plan?",
                            parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
                        }

                        sh 'terraform apply -input=false tfplan'  // Corrected action reference
                    } else if (params.action == 'destroy') {
                        sh 'terraform destroy --auto-approve'  // Corrected action reference
                    } else {
                        error "Invalid action selected. Please choose either 'apply' or 'destroy'."
                    }
                }
            }
        }
    }
}
