pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'ap-south-1'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                bat 'terraform init'
            }
        }

        stage('Terraform Apply') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'aws-credentials-1',
                    usernameVariable: 'AWS_ACCESS_KEY_ID',
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    bat 'terraform apply -auto-approve'
                }
            }
        }

        stage('Get EC2 IP') {
            steps {
                bat 'terraform output -raw ec2_ip > ip.txt'
            }
        }

        stage('Create Inventory') {
            steps {
                bat '''
                echo [web] > inventory.ini
                set /p IP=<ip.txt
                echo %IP% ansible_user=ec2-user ansible_ssh_private_key_file=small-task.pem >> inventory.ini
                '''
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                bat 'ansible-playbook -i inventory.ini nginx.yaml'
            }
        }

    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs.'
        }
    }
}
