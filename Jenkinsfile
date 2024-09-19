pipeline {
    
    agent any
    environment {
        AWS_ACCOUNT_ID=561030001202
        AWS_DEFAULT_REGION="ap-south-1" 
        IMAGE_REPO_NAME="django-notes-app"
        REPOSITORY_URI="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
        EKS_CLUSTER_NAME="kristal-eks-8PHMzFir"
        K8S_NAMESPACE="atum"
    }
    
    stages {
        stage("Clone Code") {
            steps {
                echo "Git clone"
                git url: "https://github.com/aadirai02/django-notes-app.git", branch: "main"
            }
        }

        stage("Build & Test") {
            steps {
                sh "docker build . -t django-notes-app:latest"
            }
        }

        // stage("Push to DockerHub") {
        //     steps {
        //         withCredentials([usernamePassword(
        //             credentialsId: "dockerHub", 
        //             passwordVariable: "dockerHubPass", 
        //             usernameVariable: "dockerHubUser")]) {
                    
        //             sh "docker image tag django-notes-app:latest ${dockerHubUser}/django-notes-app:${BUILD_NUMBER}"
        //             sh "docker login -u ${dockerHubUser} -p ${dockerHubPass}"
        //             sh "docker push ${dockerHubUser}/django-notes-app:${BUILD_NUMBER}"
        //         }
        //     }
        // }

        stage('Push to ECR') {
            steps {
                withCredentials([[
                $class: 'AmazonWebServicesCredentialsBinding',
                accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                secretKeyVariable: 'AWS_SECRET_ACCESS_KEY',
                credentialsId: 'dev-user-aws-credentials']]) 
                {
                    script 
                    {
                        sh 'aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID'
                        sh 'aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY'
                        sh "aws ecr get-login-password | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                        sh "docker image tag django-notes-app:latest ${REPOSITORY_URI}:${BUILD_NUMBER}"
                        sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${BUILD_NUMBER}"
                        
                    }
                }
            }
        }

        stage("Deploy") {
            steps {
                withCredentials([[
                $class: 'AmazonWebServicesCredentialsBinding',
                accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                secretKeyVariable: 'AWS_SECRET_ACCESS_KEY',
                credentialsId: 'dev-user-aws-credentials']])
                {
                    script {
                        // Update kubeconfig for EKS cluster
                        sh "aws eks --region ${AWS_DEFAULT_REGION} update-kubeconfig --name ${EKS_CLUSTER_NAME}"
                        sh 'echo "Replacing with: ${REPOSITORY_URI}:${BUILD_NUMBER}"'
                        sh 'sed -i "s|561030001202.dkr.ecr.ap-south-1.amazonaws.com/django-notes-app:\\${BUILD_NUMBER}|561030001202.dkr.ecr.ap-south-1.amazonaws.com/django-notes-app:${BUILD_NUMBER}|g" notesapp.yaml'
                        sh "/home/ubuntu/bin/kubectl apply -f notesapp.yaml --namespace ${K8S_NAMESPACE}"
                        sh "/home/ubuntu/bin/kubectl rollout status deployment/django-notes-app --namespace ${K8S_NAMESPACE}"
                }
            }
          }
        }
    }
}
