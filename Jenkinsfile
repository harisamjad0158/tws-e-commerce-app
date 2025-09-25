@Library('Shared') _

pipeline {
    agent any
    
    environment {
        // 🔹 Set DockerHub username as environment variable
        USERNAME                   = "johncorner158"

        // 🔹 Use USERNAME in image names
        DOCKER_IMAGE_NAME           = "${env.USERNAME}/easyshop-app"
        DOCKER_MIGRATION_IMAGE_NAME = "${env.USERNAME}/easyshop-migration"

        DOCKER_IMAGE_TAG            = "${BUILD_NUMBER}"
        GITHUB_CREDENTIALS          = credentials('github-credentials')
        GIT_BRANCH                  = "master"
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
                script {
                    clone("https://github.com/LondheShubham153/tws-e-commerce-app.git","master")
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
                    run_tests()
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
                script {
                    update_k8s_manifests(
                        imageTag: env.DOCKER_IMAGE_TAG,
                        manifestsPath: 'kubernetes',
                        gitCredentials: 'github-credentials',
                        gitUserName: 'Jenkins CI',
                        gitUserEmail: 'alipeco90@gmail.com'   // 🔹 updated email
                    )
                }
            }
        }

        stage('Verify Image Names') {
            steps {
                script {
                    echo "Main App Image: ${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}"
                    echo "Migration Image: ${env.DOCKER_MIGRATION_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}"
                }
            }
        }
    }
}
