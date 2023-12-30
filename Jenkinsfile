pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY  = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = "ap-northeast-3"
    }
    stages {
        stage('Checkout SCM'){
            steps{
                script{
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/batchusivaji/projects.git']])
                }
            }
        }
        stage('intializing terraform'){
            steps{
                script{
                    dir('AWS-EKS-CLUSTER'){
                        sh 'terraform init'
                    }
                }
            }
        }
        stage("alignment the terraform code"){
            steps{
                script{
                    dir('AWS-EKS-CLUSTER'){
                        sh "terraform fmt"
                    }
                }
            }
        }
        stage("validating terraform code"){
            steps{
                script{
                    dir('AWS-EKS-CLUSTER'){
                        sh "terraform validate"
                    }
                }
            }
        }
        stage('previewing terraform infra'){
            steps{
                script{
                    dir('AWS-EKS-CLUSTER'){
                        sh 'terraform plan -var-file=values.tfvars'
                    }
                    input(message: "are you sure to proceed?", ok: "proceed")
                }
            }
        }
        stage('creating/destroying EKS cluster'){
            steps{
                script{
                    dir('AWS-EKS-CLUSTER'){
                        sh 'terraform $action -var-file --auto-approve'
                    }
                }
            }
        }
        stage('deploying application'){
            steps{
                script{
                    dir('AWS-EKS-CLUSTER/manifestFiles'){
                        sh 'aws eks update-kubeconfig --name my-eks-cluster'
                        sh 'kubectl apply -f deployment.yaml'
                        sh 'kubectl apply -f service.yaml'
                    }
                }
            }
        }
    }
}