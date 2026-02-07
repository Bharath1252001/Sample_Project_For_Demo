pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "bharathg125/spaceproject"
        TAG = "v${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/Bharath1252001/Sample_Project_For_Demo.git'
            }
        }

        stage('SonarQube SAST') {
            steps {
                echo "Static Code Analysis - To be configured"
            }
        }

        stage('SCA - Dependency Check') {
            steps {
                echo "Software Composition Analysis - To be configured"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${DOCKER_IMAGE}:${TAG} .
                """
            }
        }

        stage('Trivy Image Scan') {
            steps {
                echo "Trivy scan will be configured after installation"
                // sh "trivy image ${DOCKER_IMAGE}:${TAG}"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh """
                    docker login -u $DOCKER_USER -p $DOCKER_PASS
                    docker push ${DOCKER_IMAGE}:${TAG}
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline Finished"
        }
    }
}
