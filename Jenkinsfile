pipeline {
    agent { label 'Node' }

    environment {
        AWS_REGION     = 'ap-south-1'
        AWS_ACCOUNT_ID = '769265543964'

        BACKEND_REPO  = 'wanderlust-backend'
        FRONTEND_REPO = 'wanderlust-frontend'
    }

    parameters {
        string(name: 'BACKEND_DOCKER_TAG', defaultValue: 'v1', description: 'Backend Docker Tag')
        string(name: 'FRONTEND_DOCKER_TAG', defaultValue: 'v1', description: 'Frontend Docker Tag')
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/LondheShubham153/Wanderlust-Mega-Project.git'
            }
        }

        stage('Verify Tools') {
            steps {
                sh '''
                echo "Hostname: $(hostname)"
                echo "User: $(whoami)"

                java -version
                git --version
                docker --version
                aws --version
                trivy --version

                docker images || true
                '''
            }
        }

        stage('Trivy File System Scan') {
            steps {
                sh 'trivy fs .'
            }
        }

        stage('Environment Setup') {
            parallel {

                stage('Backend Setup') {
                    steps {
                        dir('Automations') {
                            sh 'bash updatebackendnew.sh'
                        }
                    }
                }

                stage('Frontend Setup') {
                    steps {
                        dir('Automations') {
                            sh 'bash updatefrontendnew.sh'
                        }
                    }
                }
            }
        }

        stage('Build Backend Image') {
            steps {
                dir('backend') {
                    sh """
                    docker build -t ${BACKEND_REPO}:${params.BACKEND_DOCKER_TAG} .
                    """
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                dir('frontend') {
                    sh """
                    docker build -t ${FRONTEND_REPO}:${params.FRONTEND_DOCKER_TAG} .
                    """
                }
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh """
                aws ecr get-login-password --region ${AWS_REGION} | docker login \
                --username AWS \
                --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                """
            }
        }

        stage('Tag Docker Images') {
            steps {
                sh """
                docker tag ${BACKEND_REPO}:${params.BACKEND_DOCKER_TAG} \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${BACKEND_REPO}:${params.BACKEND_DOCKER_TAG}

                docker tag ${FRONTEND_REPO}:${params.FRONTEND_DOCKER_TAG} \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${FRONTEND_REPO}:${params.FRONTEND_DOCKER_TAG}
                """
            }
        }

        stage('Push Images to Amazon ECR') {
            steps {
                sh """
                docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${BACKEND_REPO}:${params.BACKEND_DOCKER_TAG}

                docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${FRONTEND_REPO}:${params.FRONTEND_DOCKER_TAG}
                """
            }
        }

    }

    post {

        success {
            echo "=================================="
            echo "CI Pipeline Completed Successfully"
            echo "=================================="
        }

        failure {
            echo "=================================="
            echo "CI Pipeline Failed"
            echo "=================================="
        }
    }
}
