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
                script {
            // Locates the installation path of your 'sonar-scanner' tool
            def scannerHome = tool 'sonar-scanner'
        // 'sonar' must match the "Name" you gave the server in the Jenkins UI
        withSonarQubeEnv('sonar') {
            sh "${scannerHome}/bin/sonar-scanner \
                -Dsonar.projectKey=frontend-pipeline \
                -Dsonar.sources=. \
                -Dsonar.host.url=http://3.83.184.106:9000"
        }
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
                sh "trivy image --severity HIGH,CRITICAL ${DOCKER_IMAGE}:${TAG}"
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
    stage('Update GitOps Manifest') {
            steps {
                script {
                    // Use the same credential ID you used for Docker Hub, 
                    // or create a new one for GitHub if needed.
                    withCredentials([usernamePassword(credentialsId: 'e80a4b04-5208-42f0-9b52-fd5a38ee2efe', 
                                     passwordVariable: 'GIT_PASSWORD', 
                                     usernameVariable: 'GIT_USERNAME')]) {
                        sh """
                        # Set git identity
                        git config --global user.email "bharathg1252001@gmail.com"
                        git config --global user.name "Bharathg1252001"
                        
                        # Clone your manifest repository
                        git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Bharath1252001/spaceproject-gitops.git
                        cd spaceproject-gitops/manifest
                        
                        # Update the image tag in deployment.yaml to the current build version
                        sed -i "s|image: bharathg125/spaceproject:.*|image: bharathg125/spaceproject:${TAG}|" deployment.yaml
                        
                        # Commit and push changes
                        git add deployment.yaml
                        git commit -m "Update image tag to ${TAG} [skip ci]"
                        git push origin main
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
