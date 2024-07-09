pipeline {
    agent any 
    tools {
        maven 'maven'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/sainath1589/devops-automation.git' 
            }
        }
        stage('Build') { 
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
                withSonarQubeEnv('sonar') { 
                    sh '''
                     sonar-scanner \
                     -Dsonar.projectKey=java \
                     -Dsonar.sources=. \
                     -Dsonar.host.url=http://35.195.28.30:9000\
                     -Dsonar.token=sqp_46e11830452f68b6402256e3d075d545219f1bd4
                    '''
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // 1. Build the Docker Image
                    sh 'docker build -t europe-west1-docker.pkg.dev/helical-button-425403-t3/java-app/my-app1 .'
                }
            }
        }
        stage('Trivy Scan') {
            steps {
                // Run Trivy analysis
                sh 'trivy image europe-west1-docker.pkg.dev/helical-button-425403-t3/java-app/my-app1:latest > trivy_report.txt'
            }
        }
     
         stage('Push Docker Image to GCR') {
            steps {
                sh 'gcloud auth configure-docker \
                    europe-west1-docker.pkg.dev'
                sh 'docker push europe-west1-docker.pkg.dev/helical-button-425403-t3/java-app/my-app1:latest'
            }
        }
           stage('Run Docker Container') {
            steps {
                script {
                    // 3. Run the Container
                    sh 'docker run -d -p 8080:8080 europe-west1-docker.pkg.dev/helical-button-425403-t3/java-app/my-app1:latest'
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
