pipeline {
    agent { label 'linux' }

    environment {
        CONTAINER_IMAGE_NAME = 'example-sonarqube-python'
        CONTAINER_IMAGE = "${CONTAINER_IMAGE_NAME}:${env.BUILD_NUMBER}" // ${env.BUILD_ID}
        CONTAINER_REGISTRY = 'your-docker-registry-url'
        K8S_DEPLOYMENT_NAME = 'example-sonarqube-python-deploy'
        K8S_NAMESPACE = 'example'
    }

    stages {
        stage('Build Image') {
            steps {
                script {
                    docker.build(CONTAINER_IMAGE)
                    //sh "docker build -t ${CONTAINER_IMAGE} ."
                    sh "ls -l"
                    sh "id"
                    sh "ping -c 3 8.8.8.8"
                    sh 'whereis sonar-scanner'
                }
            }
        }
        stage('Run Tests') {
            steps {
                script {
                    docker.image(CONTAINER_IMAGE).inside {
                        sh "python3 -m unittest test_app.py"    
                    }
                }
            }
        }

        stage('Static Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sq1') { // Specify the SonarQube server name configured in Jenkins
                        sh """
                        sonar-scanner \
                            -Dsonar.projectKey=your-project-key \
                            -Dsonar.projectName=${CONTAINER_IMAGE_NAME} \
                            -Dsonar.projectVersion=${env.BUILD_NUMBER} \
                            -Dsonar.sources=. \
                            -Dsonar.language=python # Replace with your language if different
                        """
                    }
                }
            }
        }

        /*
        stage('Push Container Image') {
            steps {
                script {
                    docker.withRegistry(REGISTRY_URL, DOCKER_CREDENTIALS_ID) {
                        docker.image(CONTAINER_IMAGE).push('latest')
                    }
                }
            }
        }

        stage('Deploy to k8s') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'kubeconfig-id', variable: 'KUBE_CONFIG')]) {
                        // Set up the Kubernetes configuration
                        sh "export KUBECONFIG=${KUBE_CONFIG}"

                        // Deploy to Kubernetes
                        sh """
                        kubectl apply -f k8s/deployment.yaml
                        kubectl set image deployment/${KUBE_DEPLOYMENT_NAME} ${KUBE_DEPLOYMENT_NAME}=${DOCKER_REGISTRY}/${DOCKER_IMAGE}-renamed
                        kubectl rollout status deployment/${KUBE_DEPLOYMENT_NAME} -n ${KUBE_NAMESPACE}
                        """
                    }
                }
            }
        }
    }
    */
    }

    post {
        success {
            echo "Pipeline completed successfully! Image pushed: ${CONTAINER_IMAGE_NAME}-renamed and deployed to Kubernetes."
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
