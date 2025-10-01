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
                cleanWs()
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: env.GIT_BRANCH, url: env.GIT_REPO, credentialsId: 'github-credentials'
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
                                                      usernameVariable: 'DOCKER_USER', 
                                                      passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    }
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

        stage('Push Docker Images') {
            parallel {
                stage('Push Main App Image') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
                                                              usernameVariable: 'DOCKER_USER', 
                                                              passwordVariable: 'DOCKER_PASS')]) {
                                sh "docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                            }
                        }
                    }
                }

                stage('Push Migration Image') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
                                                              usernameVariable: 'DOCKER_USER', 
                                                              passwordVariable: 'DOCKER_PASS')]) {
                                sh "docker push ${DOCKER_MIGRATION_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                            }
                        }
                    }
                }
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    update_k8s_manifests(
                        imageTag: env.DOCKER_IMAGE_TAG,
                        manifestsPath: 'kubernetes',
                        gitCredentials: 'github-credentials',
                        gitUserName: 'Jenkins CI',
                        gitUserEmail: 'haris.amjad@hotmail.com'
                    )
                }
            }
        }
    }
}
