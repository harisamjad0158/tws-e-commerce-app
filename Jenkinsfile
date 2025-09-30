@Library('Shared') _

pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'johncorner158'
        DOCKER_IMAGE_NAME = "${DOCKERHUB_USERNAME}/easyshop-app"
        DOCKER_MIGRATION_IMAGE_NAME = "${DOCKERHUB_USERNAME}/easyshop-migration"
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
        GIT_BRANCH = "harisamjad0158-patch-1"
        GIT_REPO = "https://github.com/harisamjad0158/tws-e-commerce-app.git"
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                script {
                    clean_ws()
                }
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build Main App Image') {
                    steps {
                        script {
                            docker_build(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                dockerfile: 'Dockerfile',
                                context: '.'
                            )
                        }
                    }
                }

                stage('Build Migration Image') {
                    steps {
                        script {
                            docker_build(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                dockerfile: 'scripts/Dockerfile.migration',
                                context: '.'
                            )
                        }
                    }
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                script {
                    docker.image('node:18.17.0').inside('-u root') {
                        sh '''
                            npm config set cache $PWD/.npm-cache --global
                            npm install --unsafe-perm
                            npm run lint
                            npm run build
                        '''
                    }
                }
            }
        }

        stage('Security Scan with Trivy') {
            steps {
                script {
                    trivy_scan()
                }
            }
        }

        stage('Push Docker Images') {
            parallel {
                stage('Push Main App Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: 'docker-hub-credentials'
                            )
                        }
                    }
                }

                stage('Push Migration Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: 'docker-hub-credentials'
                            )
                        }
                    }
                }
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    script {
                        update_k8s_manifests(
                            imageTag: env.DOCKER_IMAGE_TAG,
                            manifestsPath: 'kubernetes',
                            gitCredentials: 'github-credentials',
                            gitUserName: 'Jenkins CI',
                            gitUserEmail: 'haris.amjad@hotmail.com'
                        )

                        sh """
                            git config user.name "Jenkins CI"
                            git config user.email "haris.amjad@hotmail.com"

                            # Set remote URL with credentials
                            git remote set-url origin https://$GIT_USERNAME:$GIT_PASSWORD@github.com/harisamjad0158/tws-e-commerce-app.git

                            git fetch origin ${GIT_BRANCH}
                            git rebase origin/${GIT_BRANCH} || true

                            git add kubernetes/*
                            git commit -m "Update image tags to ${DOCKER_IMAGE_TAG} [ci skip]" || echo "No changes to commit"

                            git push origin HEAD:${GIT_BRANCH} --force-with-lease
                        """
                    }
                }
            }
        }
    }
}
