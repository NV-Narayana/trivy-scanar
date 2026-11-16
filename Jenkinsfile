pipeline {
    agent any

    tools {
        maven 'Apache-Maven'      // name from Manage Jenkins → Tools
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

        stage('Trivy Dependency Scan (pom.xml)') {
            steps {
                sh '''
                    docker run --rm \
                      -v $WORKSPACE:/project \
                      -v trivy_cache:/root/.cache \
                      $TRIVY_IMG fs \
                      --scanners vuln \
                      --skip-java-db-update \
                      --severity HIGH,CRITICAL \
                      --exit-code 0 \
                      --format table \
                      /project > trivy-fs-report.txt || true
                '''
                archiveArtifacts artifacts: 'trivy-fs-report.txt', fingerprint: true
            }
        }

        stage('Trivy Secret & Config Scan') {
            steps {
                sh '''
                    docker run --rm \
                      -v $WORKSPACE:/project \
                      -v trivy_cache:/root/.cache \
                      $TRIVY_IMG fs \
                      --scanners secret,misconfig \
                      --exit-code 0 \
                      --format table \
                      /project > trivy-secret-config-report.txt || true
                '''
                archiveArtifacts artifacts: 'trivy-secret-config-report.txt', fingerprint: true
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $APP_NAME:${BUILD_NUMBER} .'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                    docker run --rm \
                      -v /var/run/docker.sock:/var/run/docker.sock \
                      -v trivy_cache:/root/.cache \
                      $TRIVY_IMG image \
                      --scanners vuln \
                      --severity HIGH,CRITICAL \
                      --exit-code 0 \
                      --format table \
                      $APP_NAME:${BUILD_NUMBER} > trivy-image-report.txt || true
                '''
                archiveArtifacts artifacts: 'trivy-image-report.txt', fingerprint: true
            }
        }

        stage('Quality Check (Fail on CRITICAL)') {
            steps {
                sh '''
                    docker run --rm \
                      -v $WORKSPACE:/project \
                      -v trivy_cache:/root/.cache \
                      $TRIVY_IMG fs \
                      --scanners vuln \
                      --skip-java-db-update \
                      --severity CRITICAL \
                      --exit-code 1 \
                      /project
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh 'ls -lh target/'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }

    post {
        success { echo 'Pipeline executed successfully!' }
        failure { echo 'Pipeline failed. Check Console Output and Trivy reports.' }
        always  { echo 'Pipeline execution completed.' }
    }
}
