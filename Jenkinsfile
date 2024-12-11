pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment {
        SCANNER_HOME = tool 'sonar'
    }
    stages {
        stage('Git Clone') {
            steps {
                echo 'Cloning Git repository...'
                git url: 'https://github.com/sainath1589/devops-automation.git', branch: 'main'
            }
        }
        stage('Maven Build') {
            steps {
                echo 'Building the project...'
                sh 'mvn clean package'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv('sonar') {
                    sh '''
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=java \
                        -Dsonar.host.url=http://35.202.63.52:9000 \
                        -Dsonar.login=sqp_9d12f2bc2827b148986c26f38bb8f6fdd2458af3
                    '''
                }
            }
        }
        stage('Docker Image Creation') {
            steps {
                echo 'Creating Docker image...'
                sh '''
                    docker build -t us-central1-docker.pkg.dev/cranjana/java-app/my-app:latest .
                '''
            }
        }
        stage('Trivy Scan') {
            steps {
                echo 'Scanning the Docker image with Trivy...'
                sh '''
                    trivy image us-central1-docker.pkg.dev/cranjana/java-app/my-app:latest > trivy_report.txt
                '''
                echo 'Trivy scan results saved to trivy_report.txt'
            }
        }
        stage('Authenticate with Google Artifact Registry and Push Image') {
            steps {
                withCredentials([file(credentialsId: 'jenkinskey', variable: 'GCLOUD_KEYFILE_JSON')]) {
                    echo 'Authenticating with Google Cloud using service account key...'
                    sh '''
                        gcloud version
                        gcloud auth activate-service-account --key-file="$GCLOUD_KEYFILE_JSON"
                        gcloud auth configure-docker us-central1-docker.pkg.dev --quiet
                        docker images
                        docker push us-central1-docker.pkg.dev/cranjana/java-app/my-app:latest
                    '''
                }
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
