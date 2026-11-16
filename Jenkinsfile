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

        stage('Trivy Dependency Scan') {
            steps {
                sh '''
                    docker run --rm \
                      -v "$WORKSPACE:/project" \
                      -v trivy_cache:/root/.cache \
                      "$TRIVY_IMG" fs \
                      --scanners vuln \
                      --severity HIGH,CRITICAL \
                      --exit-code 0 \
                      --format table \
                      /project > trivy-fs-report.txt

                    echo "Dependency scan completed."
                '''

                archiveArtifacts artifacts: 'trivy-fs-report.txt',
                                 fingerprint: true
            }
        }

        stage('Trivy Secret & Config Scan') {
            steps {
                sh '''
                    docker run --rm \
                      -v "$WORKSPACE:/project" \
                      -v trivy_cache:/root/.cache \
                      "$TRIVY_IMG" fs \
                      --scanners secret,misconfig \
                      --exit-code 0 \
                      --format table \
                      /project > trivy-secret-config-report.txt

                    echo "Secret and configuration scan completed."
                '''

                archiveArtifacts artifacts: 'trivy-secret-config-report.txt',
                                 fingerprint: true
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    echo "Checking Dockerfile..."

                    if [ ! -f Dockerfile ]; then
                        echo "ERROR: Dockerfile not found!"
                        exit 1
                    fi

                    echo "Dockerfile found."

                    echo "Checking Maven JAR..."
                    ls -lh target/*.jar

                    echo "Building Docker image..."

                    docker build -t "$APP_NAME:$BUILD_NUMBER" .

                    echo "Docker image built successfully."

                    docker images
                '''
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                    docker run --rm \
                      -v /var/run/docker.sock:/var/run/docker.sock \
                      -v trivy_cache:/root/.cache \
                      "$TRIVY_IMG" image \
                      --scanners vuln \
                      --severity HIGH,CRITICAL \
                      --exit-code 0 \
                      --format table \
                      "$APP_NAME:$BUILD_NUMBER" > trivy-image-report.txt

                    echo "Docker image scan completed."
                '''

                archiveArtifacts artifacts: 'trivy-image-report.txt',
                                 fingerprint: true
            }
        }

        stage('Quality Check') {
            steps {
                sh '''
                    echo "Checking CRITICAL vulnerabilities..."

                    docker run --rm \
                      -v "$WORKSPACE:/project" \
                      -v trivy_cache:/root/.cache \
                      "$TRIVY_IMG" fs \
                      --scanners vuln \
                      --severity CRITICAL \
                      --exit-code 0 \
                      --format table \
                      /project > trivy-quality-report.txt

                    echo "Quality check completed."
                    echo "CRITICAL vulnerabilities are reported but will not fail this demo pipeline."
                '''

                archiveArtifacts artifacts: 'trivy-quality-report.txt',
                                 fingerprint: true
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    echo "Checking build artifacts..."
                    ls -lh target/

                    echo "Docker image:"
                    docker images "$APP_NAME"

                    echo "Deployment stage completed."
                '''

                archiveArtifacts artifacts: 'target/*.jar',
                                 fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Pipeline executed successfully!'
        }

        failure {
            echo 'FAILURE: Check Jenkins Console Output.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
