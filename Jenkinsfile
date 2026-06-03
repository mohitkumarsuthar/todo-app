pipeline {
    agent any
    
    environment {
        ECR_URI = "686382907829.dkr.ecr.ap-south-1.amazonaws.com/mohitkumar/todo-app"
        REGION = "ap-south-1"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/mohitkumarsuthar/todo-app.git'
            }
        }
        
        stage('Docker Build') {
            steps {
                sh 'docker build -t todo-app:${BUILD_NUMBER} .'
            }
        }
        
        stage('Trivy Scan') {
            steps {
                sh 'trivy image --exit-code 0 --severity HIGH,CRITICAL todo-app:${BUILD_NUMBER}'
            }
        }
        
        stage('Push to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ECR_URI}
                docker tag todo-app:${BUILD_NUMBER} ${ECR_URI}:${BUILD_NUMBER}
                docker push ${ECR_URI}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Update Deployment') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-creds', 
                                usernameVariable: 'GIT_USER', 
                                passwordVariable: 'GIT_TOKEN')]) {
                    sh '''
                    sed -i "s|todo-app:.*|todo-app:${BUILD_NUMBER}|g" deployment.yaml
                    git config user.email "jenkins@devops.com"
                    git config user.name "Jenkins"
                    git add deployment.yaml
                    git commit -m "updated image tag to ${BUILD_NUMBER}"
                    git push https://${GIT_USER}:${GIT_TOKEN}@github.com/your-username/your-repo main
                    '''
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline successful! ArgoCD will deploy automatically!'
        }
        failure {
            echo 'Pipeline failed! Check logs!'
        }
    }
}
