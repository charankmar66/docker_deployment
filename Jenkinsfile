```groovy
pipeline {
    agent any

    tools {
        // Must match the Maven installation name in Jenkins
        maven 'maven_3_10_0'
    }

    environment {
        // Replace with your actual Docker Hub username
        DOCKERHUB_USERNAME = 'chcharan42@gmail.com'
        IMAGE_NAME = 'docker-deployment'
        IMAGE_TAG = 'latest'
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/charankmar66/docker_deployment.git'
            }
        }

        stage('Build Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                      -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub_cred',
                        usernameVariable: 'DH_USERNAME',
                        passwordVariable: 'DH_PASSWORD'
                    )
                ]) {
                    sh '''
                        set +x
                        echo "$DH_PASSWORD" | docker login \
                          --username "$DH_USERNAME" \
                          --password-stdin

                        docker push \
                          ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'k8sconfigpwd',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    sh '''
                        kubectl apply -f deploymentservice.yaml
                        kubectl rollout status deployment \
                          --all --timeout=180s
                        kubectl get pods
                        kubectl get services
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Build, Docker push, and Kubernetes deployment completed.'
        }
        failure {
            echo 'Pipeline failed. Check the Jenkins console output.'
        }
    }
}