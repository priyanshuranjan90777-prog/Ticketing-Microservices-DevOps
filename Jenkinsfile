pipeline {
    agent any

    environment {
        USER = 'priyanshu907'
        TAG = "${BUILD_NUMBER}"
        KUBECONFIG = '/tmp/jenkins-kubeconfig'
    }

    stages {

        stage('Build & Push Docker Images') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'jenkins-creds',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )]) {
                    sh '''
                        echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin

                        for service in auth client orders tickets payments expiration
                        do
                            echo "Building $service..."
                            docker build -t $USER/ticketing-$service:$TAG ./$service
                            docker push $USER/ticketing-$service:$TAG
                        done
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    echo "Checking Kubernetes..."
                    kubectl get nodes

                    echo "Applying Kubernetes manifests..."
                    kubectl apply -f infra/k8s

                    echo "Updating application images..."

                    kubectl set image deployment/auth-depl \
                        auth=$USER/ticketing-auth:$TAG

                    kubectl set image deployment/client-depl \
                        client=$USER/ticketing-client:$TAG

                    kubectl set image deployment/orders-depl \
                        orders=$USER/ticketing-orders:$TAG

                    kubectl set image deployment/tickets-depl \
                        tickets=$USER/ticketing-tickets:$TAG

                    kubectl set image deployment/payments-depl \
                        payments=$USER/ticketing-payments:$TAG

                    kubectl set image deployment/expiration-depl \
                        expiration=$USER/ticketing-expiration:$TAG

                    echo "Waiting for deployments..."

                    kubectl rollout status deployment/auth-depl --timeout=180s
                    kubectl rollout status deployment/client-depl --timeout=180s
                    kubectl rollout status deployment/orders-depl --timeout=180s
                    kubectl rollout status deployment/tickets-depl --timeout=180s
                    kubectl rollout status deployment/payments-depl --timeout=180s
                    kubectl rollout status deployment/expiration-depl --timeout=180s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "===== PODS ====="
                    kubectl get pods

                    echo "===== DEPLOYMENTS ====="
                    kubectl get deployments

                    echo "===== SERVICES ====="
                    kubectl get services

                    echo "===== INGRESS ====="
                    kubectl get ingress
                '''
            }
        }
    }
}