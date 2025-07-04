pipeline {
    agent any
    environment {
        TF_IN_AUTOMATION = 'true'
        TF_CLI_CONFIG_FILE = credentials('tf-creds')
        AWS_SHARED_CREDENTIALS_FILE='/home/ubuntu/.aws/credentials'
    }
    stages {
        stage('Init') {
            steps {
                sh 'terraform init -no-color'
            }
        }
        stage('Plan') {
            steps {
                sh 'terraform plan -no-color -var-file="$BRANCH_NAME.tfvars"'
            }
        }
        stage('Validate Apply') {
            when {
                beforeInput true
                branch "dev"
            }
            input {
                message "Do you want to apply this plan?"
                ok "Apply"
            }
            steps {
                echo 'Apply Accepted'
            }
        }
        stage('Apply') {
            steps {
                sh 'terraform apply -auto-approve -no-color -var-file="$BRANCH_NAME.tfvars"'
            }
        }
        stage('EC2 Wait') {
            steps {
                 sh 'aws ec2 wait instance-status-ok --region us-west-1'
            }
        }
        stage('Validate Ansible') {
             when {
                beforeInput true
                branch "dev"
            }
            input {
                message "Run Ansible playbook?"
                ok "Sure"
            }
            steps {
                echo 'Play started'
            }
        }
        stage('Ansible') {
            steps {
                ansiblePlaybook(credentialsId: 'ec2-ssh-key', inventory: 'aws_hosts', playbook: 'playbooks/main-playbook.yml')
            }
        }
        stage('Test Grafana and Prometheus') {
          steps {
                ansiblePlaybook(credentialsId: 'ec2-ssh-key', inventory: 'aws_hosts', playbook: 'playbooks/node-test.yml') 
            }
        }
        stage('Validate Destroy') {
            input {
                message "Destroy the resources?"
                ok "Go Ahead"
            }
            steps {
                echo 'Resource termination started'
            }
        }
        stage('Destroy') {
            steps {
                sh 'terraform destroy -auto-approve -no-color -var-file="$BRANCH_NAME.tfvars"'
            }
        }
    }
    post {
      success {
        echo "Success!!!!"
      }
      failure {
        sh 'terraform destroy -auto-approve -no-color -var-file="$BRANCH_NAME.tfvars"'
      }
   }
}
