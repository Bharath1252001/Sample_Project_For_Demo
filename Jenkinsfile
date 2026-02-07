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
                echo "Static Code Analysis"
                // will configure sonar next step
            }
        }

        stage('SCA - Dependency Check') {
            steps {
                echo "Software Composition Analysis"
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
                sh """
                trivy image ${DOCKER_IMAGE}:${TAG}
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh """
                docker login -u $DOCKER_USER -p $DOCKER_PASS
                docker push ${DOCKER_IMAGE}:${TAG}
                """
            }
        }
    }

    post {
        always {
            echo "Pipeline Finished"
        }
    }
}
