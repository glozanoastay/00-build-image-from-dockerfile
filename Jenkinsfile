pipeline {
    agent { 
      dockerfile true
    }

    environment {
        CONTAINER_IMAGE_NAME = 'example-sonarqube-python'
        CONTAINER_IMAGE = "${CONTAINER_IMAGE_NAME}:${env.BUILD_NUMBER}"
        CONTAINER_REGISTRY = 'your-docker-registry-url'
        K8S_DEPLOYMENT_NAME = 'example-sonarqube-python-deploy'
        K8S_NAMESPACE = 'example'
    }

    stages {
        stage('Run Tests') {
            steps {
                script {
                    // Run tests in the Docker container
                    sh 'python -m unittest test_app.py' // Replace with your test command
                }
            }
        }

        stage('Static Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sq1') { // Specify the SonarQube server name configured in Jenkins
                        // Run SonarQube analysis
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
                    withCredentials([usernamePassword(credentialsId: 'docker-credentials-id', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        // Rename the Docker image (optional step)
                        def newImageName = "${CONTAINER_IMAGE_NAME}-renamed" // Change as needed
                        sh "docker tag ${CONTAINER_IMAGE} ${newImageName}"

                        // Login to Docker registry
                        sh """
                        echo ${DOCKER_PASSWORD} | docker login ${DOCKER_REGISTRY} -u ${DOCKER_USERNAME} --password-stdin
                        docker push ${newImageName}
                        """
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
            echo "Pipeline completed successfully! Image pushed: ${DOCKER_IMAGE}-renamed and deployed to Kubernetes."
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
