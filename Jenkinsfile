pipeline {
    agent {
    label 'ubuntu'
}

    environment {
        DOCKER_REPO = 'sujithmsuji/devops-nginx'
        IMAGE_VERSION = "1.${BUILD_NUMBER}"
        DOCKER_IMAGE = "${DOCKER_REPO}:1.${BUILD_NUMBER}"
        NAMESPACE = 'production-app'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Application') {
            steps {
                sh '''
                    echo "Verifying application files..."
                    ls -la
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE} .'
            }
        }

stage('Docker Login') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'dockerhub-creds',
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_PASSWORD'
            )
        ]) {
            sh '''
                echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USER" --password-stdin
            '''
        }
    }
}
        stage('Push Docker Image') {
            steps {
                sh 'docker push ${DOCKER_IMAGE}'
            }
        }

        stage('Prepare Kubernetes Deployment') {
            steps {
                sh '''
                    sed -i "s|IMAGE_VERSION|1.${BUILD_NUMBER}|g" deployment.yaml

                    echo "Deployment image:"
                    grep "image:" deployment.yaml
                '''
            }
        }

        stage('Deploy Kubernetes Resources') {
            steps {
                sh '''
                    kubectl apply -f configmap.yaml -n ${NAMESPACE}
                    kubectl apply -f pvc.yaml -n ${NAMESPACE}
                    kubectl apply -f deployment.yaml -n ${NAMESPACE}
                    kubectl apply -f service.yaml -n ${NAMESPACE}
                    kubectl apply -f hpa.yaml -n ${NAMESPACE}
                    kubectl apply -f vpa.yaml -n ${NAMESPACE}
                    kubectl apply -f ingress.yaml -n ${NAMESPACE}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl rollout status deployment/nginx-deployment \
                    -n ${NAMESPACE} \
                    --timeout=5m

                    echo "Deployment:"
                    kubectl get deployment nginx-deployment -n ${NAMESPACE}

                    echo "Pods:"
                    kubectl get pods -n ${NAMESPACE} -o wide

                    echo "Service:"
                    kubectl get service nginx-service -n ${NAMESPACE}

                    echo "HPA:"
                    kubectl get hpa -n ${NAMESPACE}

                    echo "PVC:"
                    kubectl get pvc -n ${NAMESPACE}

                    echo "Ingress:"
                    kubectl get ingress -n ${NAMESPACE}
                '''
            }
        }
    }
}
