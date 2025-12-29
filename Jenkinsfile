pipeline {
    agent any
    tools {
        jdk 'jdk-21'
        nodejs 'node'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                // Fixed: Explicitly using master as confirmed in your logs
                git branch: 'master', url: 'https://github.com/waseem00096/ecomerce-site.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('SonarQube') {
                    // Fixed: Removed the space after -Dsonar.projectName=
                    // Added backslash for multi-line shell command
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
                       // Fixed: Build and Push now use the same image name (ecomerce-site)
                       sh "docker build --build-arg TMDB_V3_API_KEY=0241f339597a981eef7440309193c7c5 -t waseem09/ecomerce-site:latest ."
                       sh "docker push waseem09/ecomerce-site:latest"
                    }
                }
            }
        }
        stage("TRIVY Image Scan") {
            steps {
                sh "trivy image waseem09/ecomerce-site:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                    export KUBECONFIG=/var/lib/jenkins/.kube/config
                    kubectl apply -f kubernetes/manifest.yml
                    kubectl rollout status deployment/ecomerce-deployment                    '''
                }
            }
        }
    }
    post {
        always {
            // Efficiency: Reclaims space on your Jenkins VM
            sh 'docker image prune -f'
        }
    }
}
