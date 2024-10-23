pipeline {
    agent { label 'linux' }

    environment {
        CONTAINER_IMAGE_NAME = 'glozanoastay/example-sonarqube-python' // https://hub.docker.com/r/glozanoastay/example-sonarqube-python
        CONTAINER_IMAGE_TAG = "${env.BUILD_ID}" //"${env.BUILD_NUMBER}" // ${env.BUILD_ID}
        CONTAINER_IMAGE = "${CONTAINER_IMAGE_NAME}:${CONTAINER_IMAGE_TAG}"
        CONTAINER_REGISTER_CREDS = 'jenkins-dockerhub-astay'
        CONTAINER_REGISTER_URL = "https://index.docker.io/v1/"
        K8S_DEPLOYMENT_NAME = 'example-sonarqube-python-deploy'
        K8S_NAMESPACE = 'example'
        SONARQUBE_PROJECT_KEY='00-build-image-from-dockerfile'
    }

    stages {
        stage('Build Image') {
            steps {
                script {
                    docker.build(CONTAINER_IMAGE)
                }
            }
        }
        stage('Run Tests') {
            steps {
                script {
                    docker.image(CONTAINER_IMAGE).inside {
                        sh "python3 -m unittest test_app.py"    
                    }

                    sh 'sonar-scanner -v'
                }
            }
        }

        stage('Static Analysis') {
            steps {
                script {
                    // Get the SonarScanner tool path
                    def scannerHome = tool 'SonarScanner'
                    
                    // Run the SonarScanner within the SonarQube environment
                    withSonarQubeEnv() {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }

                    sh "echo ${scannerHome}/bin/sonar-scanner"
                    sh "ls -l ${scannerHome}/bin/sonar-scanner"

                    //sh "${scannerHome}/bin/sonar-scanner -v"
                    withSonarQubeEnv(installationName: 'sq1') { // Specify the SonarQube server name configured in Jenkins
                        sh "${scannerHome}/bin/sonar-scanner -v"
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=${SONARQUBE_PROJECT_KEY} \
                            -Dsonar.projectName=${CONTAINER_IMAGE_NAME} \
                            -Dsonar.projectVersion=${env.BUILD_NUMBER} \
                            -Dsonar.sources=. \
                            -Dsonar.language=python
                        """
                    }
                }
            }
        }

        stage('Quality Gate'){
            steps {
                timeout(time: 2, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }


        stage('Push Container Image') {
            steps {
                script {
                    sh 'docker images'
                    docker.withRegistry(CONTAINER_REGISTER_URL, CONTAINER_REGISTER_CREDS) {
                        docker.image(CONTAINER_IMAGE).push()
                        docker.image(CONTAINER_IMAGE).push('latest')
                    }
                }
            }
        }

        /*
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
