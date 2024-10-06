pipeline{
    agent any
    tools{
        jdk 'Java17'
        nodejs 'NodeJS'
        maven 'Maven3'
    }
    environment {
        APP_NAME    = "myreactapp"
        RELEASE     = "1.0.0"
        DOCKER_USER = "irohitmishra"
        DOCKER_PASS = "dockerid"
        IMAGE_NAME  = "${DOCKER_USER}" + "_" + "${APP_NAME}" 
        SONAR_TOKEN = "sonarid"
        NODE_VERSION = '20.17.0'
        SONAR_SCANNER_HOME = tool 'SonarScanner'  // Name of SonarQube scanner tool in Jenkins configuration
        SONARQUBE_URL = 'http://20.121.116.30:9000/' // Your SonarQube server URL
        registryUrl = 'bayeracr.azurecr.io'
        registryCredential = 'ACR'
        ACR_NAME = 'bayeracr'
        ACR_REPO = 'bayer-myreactapp'
        IMAGE_TAG = "${env.JOB_NAME}-${env.BUILD_ID}"
        ACR_CREDENTIALS = credentials('acr-credentials')

    }
    stages{
        stage ("Clean up Workspace"){
            steps{
                echo "====== Cleaning up the Workspace ======"
                cleanWs()
            }
        }
        
        stage ("Check out from Bit Bucket Repo"){
            steps{
                echo "======Code Check out from the Main Branch ======"
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'irohitmishra-admin', url: 'https://irohitmishra-admin@bitbucket.org/irohit_hcl/myreactapp.git']])
            }
        }
        stage ("Build the Application with node"){
            steps{
                script {
                    echo "====== Now building the custom react app ======"
                    sh 'npm install'
                    sh "npm run build"
                     }
                }
        }
        stage('SonarQube Code Quality Analysis') {
            steps {
                script {
                    // Run SonarQube analysis using the SonarQube scanner
                    withSonarQubeEnv(credentialsId: 'sonarid') {  // 'SonarQube' is the SonarQube server name in Jenkins configuration
                        sh '''
                            ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                            -Dsonar.projectKey=myreactapp \
                            -Dsonar.sources=./src \
                            -Dsonar.host.url=${SONARQUBE_URL} \
                           
                        '''
                    }
                }
            }
        }
        
        stage('Quality Gate Check') {
            steps {
                script {
                    // Wait for SonarQube Quality Gate result
                        def qualityGate = waitForQualityGate()
                        if (qualityGate.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qualityGate.status}"
                        }
                    }
                }
        }
        stage('Docker Build') {
                steps {
                    sh 'docker build -t "$JOB_NAME":"${RELEASE}"."${BUILD_ID}" .'
                    sh 'docker image tag "$JOB_NAME":"${RELEASE}"."${BUILD_ID}" "${DOCKER_USER}"/"${APP_NAME}":"${RELEASE}"."${BUILD_ID}"'
                    sh 'docker image tag "$JOB_NAME":"${RELEASE}"."${BUILD_ID}" "${DOCKER_USER}"/"${APP_NAME}":latest'
                            
                    }
        }
       stage('Docker Push') {
                steps {
                withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                sh 'docker push "${DOCKER_USER}"/"${APP_NAME}":"${RELEASE}"."${BUILD_ID}"'
                sh 'docker push "${DOCKER_USER}"/"${APP_NAME}":latest'
                    }
                }
        }
       stage('Login to ACR'){
           steps{
                script{
                        // log in to Azur Container Registry
                sh "echo ${ACR_CREDENTIALS} | docker login ${ACR_NAME}.azurecr.io --username ${ACR_NAME} --password-stdin"
                }
            }
        }
        stage('Push Docker Image to ACR'){
           steps {
                script {
                        // Push the Docker Image to Container registry
                        sh "docker push ${ACR_NAME}.azurecr.io/${ACR_REPO}:${IMAGE_TAG}"
                }
            }
        }
        stage('Deploy to AKS') {
          steps {
            withKubeConfig([credentialsId: 'AKS_k8']) {
              sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"'
              sh 'chmod u+x ./kubectl'
              sh './kubectl apply -f deploymentservice.yaml'
                }
          }
        }
       stage (" Post Clean up Workspace"){
            steps{
                echo "====== Cleaning up the Workspace ======"
                cleanWs()
            }
        }
    }
}
