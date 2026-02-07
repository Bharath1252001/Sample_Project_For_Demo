pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "bharathg125/spaceproject"
        TAG = "v${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Bharath1252001/Sample_Project_For_Demo.git'
            }
        }

        stage('SonarQube SAST') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'
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
                sh "docker buildx build --load -t ${DOCKER_IMAGE}:${TAG} ."
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --severity HIGH,CRITICAL ${DOCKER_IMAGE}:${TAG}"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'e80a4b04-5208-42f0-9b52-fd5a38ee2efe', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                        sh "docker push ${DOCKER_IMAGE}:${TAG}"
                    }
                }
            }
        }

        stage('Update GitOps Manifest') {
            steps {
                script {
                    // Update this to 'github-creds' if you created the separate GitHub token credential
                    withCredentials([usernamePassword(credentialsId: 'e80a4b04-5208-42f0-9b52-fd5a38ee2efe', 
                                     passwordVariable: 'GIT_PASSWORD', 
                                     usernameVariable: 'GIT_USERNAME')]) {
                        sh """
                        git config --global user.email "bharathg1252001@gmail.com"
                        git config --global user.name "Bharathg1252001"
                        
                        # Clean up any existing directory to avoid conflicts
                        rm -rf spaceproject-gitops

                        git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Bharath1252001/spaceproject-gitops.git
                        cd spaceproject-gitops/manifest
                        
                        sed -i "s|image: bharathg125/spaceproject:.*|image: bharathg125/spaceproject:${TAG}|" deployment.yaml
                        
                        git add deployment.yaml
                        git commit -m "Update image tag to ${TAG} [skip ci]"
                        git push origin main
                        """
                    }
                }
            }
        }
    } // This is where the stages block must end

    post {
        always {
            echo "Pipeline Finished"
        }
    }
}