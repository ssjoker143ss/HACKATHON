pipeline{
    agent{
        label "jenkins-agent"
    }
    tools{
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        APP_NAME    = "java_devops_pipeleine"
        RELEASE     = "1.0.0"
        DOCKER_USER = "irohitmishra"
        DOCKER_PASS = "dockerHub"
        IMAGE_NAME  = "${DOCKER_USER}" + "_" + "${APP_NAME}"
        IMAGE_TAG   = "${RELEASE}-${BUILD_NUMBER}" 
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
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'deb0e7a2-8f46-4add-a8dd-f42875546834', url: 'https://irohitmishra-admin@bitbucket.org/irohit_hcl/java_devops_pipeleine.git']])
            }
        }
        stage ("Build the Application with Maven"){
            steps{
                echo "====== Now building the custom java application with Maven ======"
                sh "mvn clean package"
            }
        }
        stage ("SonarQube Code Analysis"){
            steps{
                script {
                    echo "====== Now doing Code Quality check via SonarQube ======"
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') 
                        {
                        sh "mvn sonar:sonar"
                        }
                }
            }
        }
        stage ("SonarQube Quality Gate Check"){
            steps{
                script {
                    echo "====== Now doing Code Quality Gate check via SonarQube ======"
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token' 
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