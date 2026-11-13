
pipeline {

    agent any

    environment {
        DOCKER_REPO = "sirsikararun"
        IMAGE_NAME = "website"
        DEPLOYMENT_NAME = "website-deployment"
        CONTAINER_NAME = "website"
    }

    stages {

        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Application') {
            steps {
                sh '''
                    test -f Dockerfile
                    test -f index.html
                    test -f deployment.yaml
                    test -f service.yaml
                    echo "Application files validated"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                    -t $DOCKER_REPO/$IMAGE_NAME:$BUILD_NUMBER .
                '''
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                        -u "$DOCKER_USER" --password-stdin

                        docker push \
                        $DOCKER_REPO/$IMAGE_NAME:$BUILD_NUMBER

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig-minikube',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    sh '''
                        kubectl apply -f k8s/deployment.yaml

                        kubectl apply -f k8s/service.yaml

                        kubectl set image \
                        deployment/$DEPLOYMENT_NAME \
                        $CONTAINER_NAME=$DOCKER_REPO/$IMAGE_NAME:$BUILD_NUMBER

                        kubectl rollout status \
                        deployment/$DEPLOYMENT_NAME \
                        --timeout=180s
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig-minikube',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    sh '''
                        kubectl get deployments
                        kubectl get pods -o wide
                        kubectl get services
                    '''
                }
            }
        }
    }
}
