pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment {
        GCLOUD_CREDS=credentials('gcloud-creds')
    }
    stages {
        stage('Checkout code') {
            steps {
                git url: "https://github.com/sainath1589/devops-automation.git", branch: "main"
            }
        }
        stage('Maven Build') { 
            steps {
                script {
                    def mavenHome = tool name: 'maven', type: 'maven'
                    def mavenCMD = "${mavenHome}/bin/mvn"
                    sh "${mavenCMD} clean package" 
                }
            }
        }
        stage('SonarQube Analysis') { 
            steps {
                sh '''
                     mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=java \
                    -Dsonar.projectName='java' \
                    -Dsonar.host.url=http://34.140.12.53:9000 \
                    -Dsonar.token=sqp_8bc689bdd1289d2e425ddb477dd0bb94ebf95ef3
                    '''
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // 1. Build the Docker Image
                    sh 'docker build -t europe-west1-docker.pkg.dev/oval-cyclist-426414-p0/java-app/my-app1:v1 .'
                }
            }
        }
        stage('Trivy Scan') {
            steps {
                // Run Trivy analysis
                sh 'trivy image europe-west1-docker.pkg.dev/oval-cyclist-426414-p0/java-app/my-app1:latest > trivy_report.txt'
            }
        }
        stage('Push Image to Artifact Registry') {
            steps {
                withCredentials([file(credentialsId: 'gcloud-creds', variable: 'GCLOUD_CREDS')]) {
                    sh '''
                    gcloud version
                    gcloud auth activate-service-account --key-file="$GCLOUD_CREDS"
                    gcloud auth configure-docker europe-west1-docker.pkg.dev
                    docker push europe-west1-docker.pkg.dev/oval-cyclist-426414-p0/java-app/my-app1:v1
                    
                    '''
                }
            }
        }
        
    }
   post {
        success {
            // Actions to take if the pipeline succeeds
            echo 'Pipeline succeeded!'
            // You can also send an email notification on success
            mail to: 'sainathreddy250@gmail.com',
                 subject: "Pipeline Succeeded: ${currentBuild.fullDisplayName}",
                 body: "The pipeline ${env.BUILD_URL} has successfully completed."
        }
        failure {
            // Actions to take if the pipeline fails
            echo 'Pipeline failed!'
            // You can also send an email notification on failure
            mail to: 'sainathreddy250@gmail.com',
                 subject: "Pipeline Failed: ${currentBuild.fullDisplayName}",
                 body: "The pipeline ${env.BUILD_URL} has failed. Check the logs for details."
        }
    }
}
