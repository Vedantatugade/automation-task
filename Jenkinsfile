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
                sh '''
                terraform init
                '''
            }
        }

        stage('Terraform Apply') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'aws-credentials-1',
                    usernameVariable: 'AWS_ACCESS_KEY_ID',
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    terraform apply -auto-approve
                    '''
                }
            }
        }

        stage('Get EC2 IP') {
            steps {
                sh '''
                terraform output -raw ec2_ip > ip.txt
                '''
            }
        }

        stage('Create Inventory') {
            steps {
                sh '''
                echo "[web]" > inventory.ini
                IP=$(cat ip.txt)
                echo "$IP ansible_user=ec2-user ansible_ssh_private_key_file=small-task.pem" >> inventory.ini
                '''
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                sh '''
                ansible-playbook -i inventory.ini nginx.yaml
                '''
            }
        }

    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }
}
