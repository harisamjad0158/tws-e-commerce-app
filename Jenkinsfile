pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'johncorner158'
        DOCKER_IMAGE_NAME = "${DOCKERHUB_USERNAME}/easyshop-app"
        DOCKER_MIGRATION_IMAGE_NAME = "${DOCKERHUB_USERNAME}/easyshop-migration"
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
        GIT_REPO = "https://github.com/harisamjad0158/tws-e-commerce-app.git"
        GIT_BRANCH = "harisamjad0158-patch-1"
        GIT_USER = "harisamjad0158"
        GIT_TOKEN = "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"
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: "${GIT_BRANCH}",
                    url: "https://${GIT_USER}:${GIT_TOKEN}@github.com/harisamjad0158/tws-e-commerce-app.git"
            }
        }

        stage('Docker Login') {
            steps {
                sh """
                    echo "Logging in to Docker Hub..."
                    echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                """
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build App Image') {
                    steps {
                        sh """
                            docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} -f Dockerfile .
                        """
                    }
                }

                stage('Build Migration Image') {
                    steps {
                        sh """
                            docker build -t ${DOCKER_MIGRATION_IMAGE_NAME}:${DOCKER_IMAGE_TAG} -f scripts/Dockerfile.migration .
                        """
                    }
                }
            }
        }

        stage('Push Docker Images') {
            parallel {
                stage('Push App Image') {
                    steps {
                        sh "docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                    }
                }
                stage('Push Migration Image') {
                    steps {
                        sh "docker push ${DOCKER_MIGRATION_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    echo "Updating Kubernetes manifests with image tag: ${DOCKER_IMAGE_TAG}"
                    
                    sh """
                        git config user.name "Jenkins CI"
                        git config user.email "haris.amjad@hotmail.com"
                        git remote set-url origin https://${GIT_USER}:${GIT_TOKEN}@github.com/harisamjad0158/tws-e-commerce-app.git
                        
                        sed -i "s|image:.*easyshop-app:.*|image: ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}|g" kubernetes/08-easyshop-deployment.yaml
                        sed -i "s|image:.*easyshop-migration:.*|image: ${DOCKER_MIGRATION_IMAGE_NAME}:${DOCKER_IMAGE_TAG}|g" kubernetes/12-migration-job.yaml
                        
                        git add kubernetes/*
                        git commit -m "Update image tags to ${DOCKER_IMAGE_TAG} [ci skip]" || echo "No changes to commit"
                        git push origin HEAD:${GIT_BRANCH} --force-with-lease
                    """
                }
            }
        }
    }
}
