pipeline {
    agent any
    tools {
        jdk 'jdk-21'
        nodejs 'node'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        // Define your Docker image name here for consistency
        IMAGE_NAME = "waseem09/ecomerce-site"
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'master', url: 'https://github.com/waseem00096/ecomerce-site.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectName=ecomerce-site \
                    -Dsonar.projectKey=ecomerce-site"
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install --legacy-peer-deps"
            }
        }        
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push") {
            steps {
                script {
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {   
                       // Build using the specific Jenkins Build Number for unique tagging
                       sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
                       sh "docker build -t ${IMAGE_NAME}:latest ."
                       sh "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
                       sh "docker push ${IMAGE_NAME}:latest"
                    }
                }
            }
        }
        stage("TRIVY Image Scan") {
            steps {
                sh "trivy image ${IMAGE_NAME}:latest > trivyimage.txt" 
            }
        }
        stage('Update Git Manifest') {
            steps {
                script {
                    // Update the kubernetes/manifest.yml with the new image tag
                    withCredentials([usernamePassword(credentialsId: 'github-token', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh """
                        git config user.email "jenkins@example.com"
                        git config user.name "Jenkins-CI"
                        
                        # Replace the image tag with the specific build number
                        sed -i 's|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${BUILD_NUMBER}|g' kubernetes/manifest.yml
                        
                        git add kubernetes/manifest.yml
                        git commit -m "Update image tag to ${BUILD_NUMBER} [skip ci]"
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/waseem00096/ecomerce-site.git master
                        """
                    }
                }
            }
        }
    }
    post {
        always {
            // Reclaims space on your Jenkins VM
            sh 'docker image prune -f'
        }
    }
}
