pipeline {

    agent any

    environment {
        DOCKER_IMAGE = "joshdoc/node-app"
        DOCKER_TAG = "latest"
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image $DOCKER_IMAGE:$DOCKER_TAG"
                sh "docker build -t $DOCKER_IMAGE:$DOCKER_TAG ."
            }
        }

        stage('Login Docker') {
            steps {
                echo "Logging into Docker registry"
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                }
            }
        }

        stage('Push Image') {
            steps {
                echo "Pushing Docker image $DOCKER_IMAGE:$DOCKER_TAG"
                sh "docker push $DOCKER_IMAGE:$DOCKER_TAG"
            }
        }

        stage('Debug Kubeconfig (Temporary)') {
            steps {
                echo "Verifying flattened kubeconfig for Jenkins"
                withCredentials([file(
                    credentialsId: 'kubeconfig',
                    variable: 'KUBECONFIG'
                )]) {
                    sh '''
                        echo "KUBECONFIG path: $KUBECONFIG"
                        echo "---- Checking for ubuntu paths ----"
                        grep ubuntu $KUBECONFIG || echo "No ubuntu paths found"
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying to Kubernetes cluster"
                withCredentials([file(
                    credentialsId: 'kubeconfig',
                    variable: 'KUBECONFIG'
                )]) {
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    sh 'kubectl apply -f k8s/service.yaml'
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished"
        }
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
