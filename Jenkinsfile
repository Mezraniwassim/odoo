pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS = credentials('azure-service-principal') // Azure service principal credentials stored in Jenkins
        KUBECONFIG = credentials('kubeconfig') // Kubeconfig file stored in Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the repository containing the YAML and Terraform files
                git 'https://github.com/Mezraniwassim/odoo.git'
            }
        }

        stage('Login to Azure') {
            steps {
                script {
                    withCredentials([azureServicePrincipal(credentialsId: 'azure-service-principal')]) {
                        sh '''
                        az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID
                        '''
                    }
                }
            }
        }

        stage('Terraform Init') {
            steps {
                script {
                    dir('terraform') {
                        sh 'terraform init'
                    }
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                script {
                    dir('terraform') {
                        sh 'terraform apply -auto-approve'
                    }
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        sh 'kubectl apply -f kubernetes/postgresql-deployment.yaml'
                        sh 'kubectl apply -f kubernetes/postgresql-service.yaml'
                        sh 'kubectl apply -f kubernetes/postgresql-pvc.yaml'
                        sh 'kubectl apply -f kubernetes/odoo-deployment.yaml'
                        sh 'kubectl apply -f kubernetes/odoo-service.yaml'
                    }
                }
            }
        }
    }

    post {
        always {
            node {
                cleanWs()
            }
        }
    }
}
