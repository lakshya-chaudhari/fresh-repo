pipeline {
    agent any

    environment {
        AWS_REGION      = 'ap-southeast-1'
        ECR_REPO        = '917246556423.dkr.ecr.ap-southeast-1.amazonaws.com/simple-page'
        GIT_USER_NAME   = 'lakshya-chaudhari'
        MANIFESTS_REPO  = "https://github.com/${GIT_USER_NAME}/myproject.git"
        IMAGE_TAG       = "${env.GIT_COMMIT.take(7)}"
    }

    stages {
        stage('Checkout App Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('app') {
                    sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${ECR_REPO}
                    docker push ${ECR_REPO}:${IMAGE_TAG}
                """
            }
        }

        stage('Update Manifests Repo') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-creds',
                                  usernameVariable: 'GIT_USER',
                                  passwordVariable: 'GIT_TOKEN')]) {
                    sh """
                        rm -rf manifests-repo
                        git clone https://\${GIT_USER}:\${GIT_TOKEN}@github.com/${GIT_USER_NAME}/myproject.git manifests-repo
                        cd manifests-repo
                        sed -i "s|image: .*|image: ${ECR_REPO}:${IMAGE_TAG}|" deployment.yaml
                        git config user.email "jenkins@ci.local"
                        git config user.name "Jenkins CI"
                        git add deployment.yaml
                        git commit -m "Update image to ${IMAGE_TAG} [ci skip]"
                        git push origin main
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline succeeded — image ${IMAGE_TAG} pushed to ECR (ap-southeast-1) and myproject updated. ArgoCD will sync shortly."
        }
        failure {
            echo "Pipeline failed — check logs above."
        }
    }
}
