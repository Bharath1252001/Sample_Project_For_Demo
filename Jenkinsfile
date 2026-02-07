pipeline {
    agent any
    tools {
        // This name MUST match exactly the Name you gave in Step 1
        // and it should also include a JDK if your scanner requires it
        'hudson.plugins.sonar.SonarRunnerInstallation': 'sonar-scanner'
    }

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
        // 'sonar' must match the "Name" you gave the server in the Jenkins UI
        withSonarQubeEnv('sonar') {
            sh "sonar-scanner \
                -Dsonar.projectKey=frontend-pipeline \
                -Dsonar.sources=. \
                -Dsonar.host.url=http://3.83.184.106:9000"
        }
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
                docker buildx build --load -t ${DOCKER_IMAGE}:${TAG} .
                """
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --severity HIGH,CRITICAL ${DOCKER_IMAGE}:${TAG}'
            }
        }

        stage('Pre-Push Debug') {
            steps {
                sh """
                echo "Current user:"
                whoami

                echo "Available images:"
                docker images
                """
            }
        }

        stage('Push to Docker Hub') {
    steps {
        script {
            // This securely logs you in and out automatically
            withCredentials([usernamePassword(credentialsId: 'e80a4b04-5208-42f0-9b52-fd5a38ee2efe', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                sh "docker push ${DOCKER_IMAGE}:${TAG}"
            }
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
