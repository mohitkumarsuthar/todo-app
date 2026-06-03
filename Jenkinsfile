pipeline {
    agent any
    
    environment {
        ECR_URI = "686382907829.dkr.ecr.ap-south-1.amazonaws.com/mohitkumar/todo-app"
        REGION = "ap-south-1"
    }
    
    stages {
        stage('Checkout') {
            steps {
                // changelog और poll को false करने से यह स्टेज लूप को रोकने में मदद करती है
                git branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/mohitkumarsuthar/todo-app.git',
                    changelog: false, 
                    poll: false
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
                withCredentials([usernamePassword(credentialsId: 'github', 
                                usernameVariable: 'GIT_USER', 
                                passwordVariable: 'GIT_TOKEN')]) {
                    sh '''
                    # 1. रिपॉजिटरी को क्लीन और लेटेस्ट रखना ताकि पुश फेल न हो
                    git fetch origin main
                    git checkout main
                    
                    # 2. क्रेडेंशियल्स को मास्क करके ही पुश यूआरएल सेट करना
                    git remote set-url origin https://${GIT_USER}:${GIT_TOKEN}@github.com/mohitkumarsuthar/todo-app.git
                    
                    # 3. इमेज टैग अपडेट करना
                    sed -i "s|todo-app:.*|todo-app:${BUILD_NUMBER}|g" deployment.yaml
                    
                    git config user.email "jenkins@devops.com"
                    git config user.name "Jenkins"
                    
                    git add deployment.yaml
                    
                    # 4. कमिट मैसेज में [skip ci] और [ci skip] दोनों ऐड करना ताकि अलग-अलग Git Providers इसे पहचान सकें
                    git commit -m "updated image tag to ${BUILD_NUMBER} [skip ci] [ci skip]"
                    
                    # 5. पुश करना
                    git push origin main
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
