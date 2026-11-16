pipeline {
    agent any

    tools {
        maven 'Apache-Maven'
    }

    environment {
        APP_NAME  = 'trivy-demo'
        TRIVY_IMG = 'aquasec/trivy:latest'
    }

    stages {

        stage('Checkout Git') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/NV-Narayana/trivy-scanar.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Trivy Code Scan') {
            steps {
                sh 'docker run --rm -v $WORKSPACE:/project aquasec/trivy:latest fs --scanners vuln,secret,misconfig --skip-java-db-update /project > trivy-fs-report.txt || true'
                archiveArtifacts artifacts: 'trivy-fs-report.txt', allowEmptyArchive: true
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $APP_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --scanners vuln --skip-java-db-update --severity HIGH,CRITICAL $APP_NAME:$BUILD_NUMBER > trivy-image-report.txt || true'
                archiveArtifacts artifacts: 'trivy-image-report.txt', allowEmptyArchive: true
            }
        }
    }

    post {
        always { echo 'Pipeline completed.' }
        success { echo 'Pipeline succeeded.' }
        failure { echo 'Pipeline failed. Check Console Output.' }
    }
}
