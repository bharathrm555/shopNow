pipeline {
    agent any

    environment {
        NAMESPACE = 'shopnow'
        BACKEND_IMAGE = 'backend:latest'
        FRONTEND_IMAGE = 'frontend:latest'
        ADMIN_IMAGE = 'admin:latest'
        CHART_PATH = './charts/shopnow'
        KUBECONFIG = "${env.HOME}/.kube/config"
    }

    stages {
        stage('Tool Check') {
            steps {
                script {
                    sh 'docker --version'
                    sh 'kubectl version --client'
                    sh 'helm version'
                    sh 'kind version'
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    echo 'Building Backend Image...'
                    sh "docker build -t ${BACKEND_IMAGE} ./backend"
                    
                    echo 'Building Frontend Image...'
                    sh "docker build -t ${FRONTEND_IMAGE} ./frontend"
                    
                    echo 'Building Admin Image...'
                    sh "docker build -t ${ADMIN_IMAGE} ./admin"
                }
            }
        }

        stage('Load Images to KIND') {
            steps {
                script {
                    echo 'Loading images into KIND cluster...'
                    sh "kind load docker-image ${BACKEND_IMAGE}"
                    sh "kind load docker-image ${FRONTEND_IMAGE}"
                    sh "kind load docker-image ${ADMIN_IMAGE}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo 'Deploying with Helm...'
                    // Ensure namespace exists first (or let helm handle it with --create-namespace)
                    sh "kubectl create namespace ${NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -"
                    
                    sh "helm upgrade --install shopnow ${CHART_PATH} --namespace ${NAMESPACE}"
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    echo 'Verifying Pods...'
                    sh "kubectl get pods -n ${NAMESPACE}"
                    sh "kubectl get svc -n ${NAMESPACE}"
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed. Check logs.'
        }
    }
}
