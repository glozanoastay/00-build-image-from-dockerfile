pipeline {
    //agent { dockerfile true }
    agent { label 'linux' }

    parameters {
        string(name: 'IMAGE_NAME', defaultValue: 'example-python', description: 'Container Image Name')
        string(name: 'IMAGE_REGISTRY_URI', defaultValue: 'https://720766170633.dkr.ecr.us-east-2.amazonaws.com', description: 'Container Image Registry')

        /*
        // WITHOUT SonarQube PLUGIN
        string(name: 'SONAR_PROJECT', defaultValue: 'example02', description: 'SonarQube Project Key')
        string(name: 'SONAR_HOST', defaultValue: 'http://localhost:9000', description: 'SonarQube Host')
        */
    }

    stages {
        stage('Build Container Image') {
            steps {
                script {
                    def imageName = "${params.IMAGE_NAME}"
                    image = docker.build("${imageName}")
                }
            }
        }

        stage('Run Code Analysis') {
            steps {
                withSonarQubeEnv(installationName: "sq"){
                    sh 'sonar-scanner'
                }
                /*
                // WITHOUT SonarQube PLUGIN
                withCredentials([string(credentialsId: 'example-sonarcube-creds', variable: 'SONAR_TOKEN')]) {
                    sh '''
                    sonar-scanner \
                        -Dsonar.projectKey=${params.SONAR_PROJECT} \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${params.SONAR_HOST} \
                        -Dsonar.token=$SONAR_TOKEN
                    '''
                }
                */
            }
        }

        stage ("Quality Gate"){ // wait for sonarqube analysis and abort execution if source code does not meet required quality level
            steps {
                timeout(time: 2, unit: "MINUTES"){
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Push Container Image') {
            steps {
                script {
                    docker.withRegistry("${params.IMAGE_REGISTRY_URI}", 'ecr:us-east-2:aws-credentials') {
                        image.push("${env.BUILD_NUMBER}")
                        image.push("latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'kubectl apply -f k8s/deployment.yaml'
                }
            }
        }
    }
}

