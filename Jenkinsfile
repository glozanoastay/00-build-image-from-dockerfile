pipeline {
    //agent { dockerfile true }
    agent { label 'linux' }

    parameters {
        string(name: 'IMAGE_NAME', defaultValue: 'example-python', description: 'Container Image Name')
        //string(name: 'IMAGE_REGISTRY_URI', defaultValue: 'https://720766170633.dkr.ecr.us-east-2.amazonaws.com', description: 'Container Image Registry')

        // WITHOUT SonarQube PLUGIN
        string(name: 'SONAR_PROJECT', defaultValue: '', description: 'SonarQube Project Key')
        string(name: 'SONAR_HOST', defaultValue: 'http://localhost:9000', description: 'SonarQube Host')
    }

    stages {
        stage("Init"){
            steps {
                sh 'pwd'
                sh 'ls'
            }
        }

        stage('Run Code Analysis') {
            steps {
                sh 'whereis sonar-scanner'
                sh 'sonar-scanner --help'
                /*
                withSonarQubeEnv(installationName: "sq1"){
                    sh 'sonar-scanner'
                }
                */
                sh '''
                sonar-scanner \
                -Dsonar.projectKey=00-build-image-from-dockerfile \
                -Dsonar.sources=. \
                -Dsonar.host.url=http://192.168.56.1:9000 \
                -Dsonar.token=squ_37f5b383156684059f788c6eee6c305823dad0d6
                '''
                
                /*
                withSonarQubeEnv(installationName: "sq"){
                    sh 'sonar-scanner'
                }
                */
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



        /*
        stage('Build Container Image') {
            steps {
                script {
                    def imageName = "${params.IMAGE_NAME}"
                    image = docker.build("${imageName}")
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
    */
    }
}

