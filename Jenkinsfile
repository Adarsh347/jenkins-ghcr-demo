pipeline {

    agent any

    environment {
        APP_NAME = 'jenkins-ghcr-demo'
        GHCR_IMAGE = 'ghcr.io/adarsh347/jenkins-ghcr-demo'
        GHCR_CREDENTIALS = 'ghcr-credentials'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Cloning application source code...'

                git(
                    branch: 'main',
                    url: 'https://github.com/Adarsh347/jenkins-ghcr-demo.git'
                )
            }
        }

        stage('Test') {
            steps {
                echo 'Running automated tests...'

                sh 'mvn clean test'
            }
        }

        stage('Build Application') {
            steps {
                echo 'Building Maven application...'

                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'

                sh """
                    docker build -t ${GHCR_IMAGE}:latest .
                """
            }
        }

        stage('Tag Docker Image') {
            steps {
                echo 'Tagging Docker image...'

                sh """
                    docker tag ${GHCR_IMAGE}:latest \
                    ${GHCR_IMAGE}:build-${BUILD_NUMBER}
                """
            }
        }

        stage('Push Image to GHCR') {
            steps {
                echo 'Logging in to GitHub Container Registry...'

                withCredentials([
                    usernamePassword(
                        credentialsId: "${GHCR_CREDENTIALS}",
                        usernameVariable: 'GITHUB_USERNAME',
                        passwordVariable: 'GITHUB_TOKEN'
                    )
                ]) {

                    sh '''
                        echo "$GITHUB_TOKEN" | docker login ghcr.io \
                        -u "$GITHUB_USERNAME" \
                        --password-stdin
                    '''

                    sh """
                        docker push ${GHCR_IMAGE}:latest
                        docker push ${GHCR_IMAGE}:build-${BUILD_NUMBER}
                    """
                }
            }
        }
    }

    post {

        success {
            echo 'Pipeline completed successfully!'

            mail(
                to: 'adarshchandran6162@gmail.com',
                subject: "SUCCESS: Jenkins Build #${BUILD_NUMBER}",
                body: """
Hello,

The Jenkins pipeline completed successfully.

Application: ${APP_NAME}
Build Number: ${BUILD_NUMBER}

Docker Image:
${GHCR_IMAGE}:build-${BUILD_NUMBER}

Status: SUCCESS

Regards,
Jenkins
"""
            )
        }

        failure {
            echo 'Pipeline failed!'

            mail(
                to: 'YOUR_EMAIL@example.com',
                subject: "FAILED: Jenkins Build #${BUILD_NUMBER}",
                body: """
Hello,

The Jenkins pipeline has FAILED.

Application: ${APP_NAME}
Build Number: ${BUILD_NUMBER}

Please check the Jenkins console output.

Build URL:
${BUILD_URL}

Status: FAILED

Regards,
Jenkins
"""
            )
        }

        always {
            sh '''
                docker logout ghcr.io || true
            '''
        }
    }
}
