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
      stage('Blue-Green Deploy') {
    steps {
        script {
            withCredentials([usernamePassword(credentialsId: 'github-token', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                sh '''
                    git config user.email "jenkins@example.com"
                    git config user.name "Jenkins-CI"
                    git pull origin master --rebase

                    # 1. Check which version is currently LIVE in the service
                    CURRENT_LIVE=$(grep "version:" kubernetes/manifest.yml | tail -n 1 | awk '{print $2}')
                    
                    if [ "$CURRENT_LIVE" == "blue" ]; then
                        TARGET="green"
                    else
                        TARGET="blue"
                    fi
                    
                    echo "Current live is $CURRENT_LIVE. Deploying to $TARGET."

                    # 2. Update the IMAGE for the TARGET deployment only
                    # This uses a line-specific sed to find the ecomerce-TARGET deployment block
                    # Note: This assumes your manifest.yml has blue first then green
                    if [ "$TARGET" == "blue" ]; then
                        # Update image for ecomerce-blue
                        sed -i "/name: ecomerce-blue/,/image:/ s|image: .*|image: ${IMAGE_NAME}:${BUILD_NUMBER}|" kubernetes/manifest.yml
                    else
                        # Update image for ecomerce-green
                        sed -i "/name: ecomerce-green/,/image:/ s|image: .*|image: ${IMAGE_NAME}:${BUILD_NUMBER}|" kubernetes/manifest.yml
                    fi

                    # 3. Switch the Service Selector to the new TARGET
                    sed -i "s|version: $CURRENT_LIVE|version: $TARGET|g" kubernetes/manifest.yml

                    git add kubernetes/manifest.yml
                    git commit -m "Switch traffic to $TARGET with image v${BUILD_NUMBER} [skip ci]" || echo "No changes"
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
