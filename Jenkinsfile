pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'eswarpala1988'
        IMAGE_NAME     = 'my-app'
        GITOPS_REPO    = 'github.com/eswar1030/my-app-gitops.git'
        COMMIT_SHA     = "${env.GIT_COMMIT.take(7)}"
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                checkout scm
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        // Log in to Docker Hub
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                        
                        // Build Docker Image
                        sh "docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${COMMIT_SHA} ."
                        
                        // Push Docker Image
                        sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${COMMIT_SHA}"
                    }
                }
            }
        }

        stage('Update GitOps Repository') {
            steps {
                script {
                    withCredentials([string(
                        credentialsId: 'gitops-pat',
                        variable: 'GITHUB_PAT'
                    )]) {
                        dir('gitops-workspace') {
                            // Clone GitOps repo using PAT
                            sh "git clone https://${GITHUB_PAT}@${GITOPS_REPO} ."
                            
                            // Update deployment.yaml image tag
                            sh "sed -i 's|image: .*|image: ${DOCKERHUB_USER}/${IMAGE_NAME}:${COMMIT_SHA}|g' deployment.yaml"
                            
                            // Commit and push changes back to my-app-gitops
                            sh '''
                                git config user.name "jenkins-bot"
                                git config user.email "jenkins@example.com"
                                git commit -am "chore: update image tag to ${COMMIT_SHA} [jenkins]"
                                git push origin main
                            '''
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            // Cleanup Docker image and temporary workspace directory
            sh "docker rmi ${DOCKERHUB_USER}/${IMAGE_NAME}:${COMMIT_SHA} || true"
            dir('gitops-workspace') {
                deleteDir()
            }
        }
    }
}
