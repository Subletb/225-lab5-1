pipeline {
    agent any 

    environment {
        DOCKER_CREDENTIALS_ID = 'roseaw-dockerhub'
        DOCKER_IMAGE = 'cithit/subletb-lab5'   // ✅ FIXED IMAGE NAME
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        GITHUB_URL = 'https://github.com/Subletb/225-lab5-1.git'
        KUBECONFIG = credentials('subletb-225') // Jenkins stored kubeconfig
    }

    stages {

        stage('Checkout') {
            steps {
                cleanWs()
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: "${GITHUB_URL}"]]
                ])
            }
        }

        stage('Static Code Testing') {
            steps {
                // Use compatible version for Jenkins node
                sh 'npm install htmlhint@1.8.0 --save-dev'
                sh 'npx htmlhint index.html'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                
                    writeFile file: 'kubeconfig', text: KUBECONFIG

                    sh """
                    export KUBECONFIG=\$PWD/kubeconfig
                    sed -i 's|image: .*|image: ${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment-dev.yaml
                    kubectl apply -f deployment-dev.yaml
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh """
                export KUBECONFIG=\$PWD/kubeconfig
                kubectl get pods
                kubectl get services
                kubectl get deployments
                """
            }
        }
    }

    post {
        success {
            slackSend color: "good", message: "SUCCESS: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
        failure {
            slackSend color: "danger", message: "FAILED: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
    }
}
