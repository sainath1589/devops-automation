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
                      mvn clean verify sonar:sonar \
                     -Dsonar.projectKey=app2 \
                     -Dsonar.host.url=http://35.205.162.228:9000 \
                     -Dsonar.login=afca5b91a6ca5498940473a2774fa2c0efc409e3
                    '''
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // 1. Build the Docker Image
                    sh 'docker build -t sainath15890/my-app1:latest .'
                }
            }
        }
        stage('Trivy Scan') {
            steps {
                // Run Trivy analysis
                sh 'trivy image sainath15890/my-app1:latest > trivy_report.txt'
            }
        }
     
         stage('Push Docker Image to Docker Hub') {
            steps {
                sh 'docker login -u sainath15890 -p 123abc456'
                sh 'docker push sainath15890/my-app1:latest'
            }
        }
           stage('Run Docker Container') {
            steps {
                script {
                    // 3. Run the Container
                    sh 'docker run -d -p 8085:8085 sainath15890/my-app1:latest'
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
