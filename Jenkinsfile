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
    stage('Argo Rollout Deploy') {
    steps {
        script {
            withCredentials([usernamePassword(credentialsId: 'github-token', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                sh '''
                    git config user.email "jenkins@example.com"
                    git config user.name "Jenkins-CI"
                    git pull origin master --rebase

                    # Update the image in the Rollout manifest
                    sed -i "s|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${BUILD_NUMBER}|g" kubernetes/rollout.yml

                    git add kubernetes/rollout.yml
                    git commit -m "Rollout version ${BUILD_NUMBER} [skip ci]" || echo "No changes"
                    git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/waseem00096/ecomerce-site.git master
                '''
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
