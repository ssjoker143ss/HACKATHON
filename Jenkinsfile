pipeline{
    agent any
    tools{
        jdk 'Java17'
        nodejs 'nodejs'
        maven 'Maven3'
    }
    environment {
        APP_NAME    = "myreactapp"
        RELEASE     = "1.0.0"
        DOCKER_USER = "irohitmishra"
        DOCKER_PASS = "dockerHub"
        IMAGE_NAME  = "${DOCKER_USER}" + "_" + "${APP_NAME}"
        IMAGE_TAG   = "${RELEASE}-${BUILD_NUMBER}" 
        SONAR_TOKEN = "jenkins-sonar-token"
        NODE_VERSION = '20.17.0'
        SONAR_SCANNER_HOME = tool 'SonarQubeScanner'  // Name of SonarQube scanner tool in Jenkins configuration
        SONARQUBE_URL = 'http://your-sonarqube-server-url' // Your SonarQube server URL
        SONARQUBE_AUTH_TOKEN = credentials('sonarqube-auth-token')  // SonarQube token stored in Jenkins credentials
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
        stage('SonarQube Analysis') {
            steps {
                script {
                    // Run SonarQube analysis using the SonarQube scanner
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {  // 'SonarQube' is the SonarQube server name in Jenkins configuration
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
        
        stage('Quality Gate') {
            steps {
                script {
                    // Wait for SonarQube Quality Gate result
                    timeout(time: 1, unit: 'HOURS') {
                        def qualityGate = waitForQualityGate()
                        if (qualityGate.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qualityGate.status}"
                        }
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
       stage('Deploy to Minikube') {
        steps {
            sh 'kubectl apply -f deploymentservice.yaml'
            }
       }
    }
}
