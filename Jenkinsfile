pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-cred'
        DOCKER_IMAGE          = 'Johnjose10/fast-api-chatbot'
        SONAR_SERVER_NAME     = 'sonarqube-env' 
        SONAR_SCANNER_TOOL    = 'sonar-scanner'     
        GIT_CREDENTIALS_ID    = 'github-cred'
pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-cred'
        DOCKER_IMAGE          = 'docker777john/fast-api-chatbot'

        SONAR_SERVER_NAME     = 'sonarqube-env'
        SONAR_SCANNER_TOOL    = 'sonar-scanner'

        GIT_CREDENTIALS_ID    = 'github-cred'
        MANIFEST_REPO         = 'https://github.com/Johnjose10/fast-api-chatbot.git'
        MANIFEST_BRANCH       = 'main'
    }

    stages {

        // 1. Checkout source code
        stage('Checkout Source Code') {
            steps {
                echo 'Checking out source code from GitHub...'
                checkout scm
            }
        }

        // 2. SonarQube analysis
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool "${SONAR_SCANNER_TOOL}"

                    withSonarQubeEnv("${SONAR_SERVER_NAME}") {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=fast-api-chatbot-key \
                            -Dsonar.projectName=fast-api-chatbot-project \
                            -Dsonar.sources=. \
                            -Dsonar.exclusions=**/venv/**,**/.venv/**,**/__pycache__/**,**/.pytest_cache/**,**/static/**
                        """
                    }
                }
            }
        }

        // 3. Build and push Docker image
        stage('Build & Push Docker Image') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: "${DOCKER_CREDENTIALS_ID}",
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )
                    ]) {
                        sh '''
                            set -e

                            echo "$DOCKER_PASS" | docker login \
                                -u "$DOCKER_USER" \
                                --password-stdin

                            echo "Building Docker image..."

                            docker build \
                                -t "$DOCKER_IMAGE:$BUILD_NUMBER" \
                                -t "$DOCKER_IMAGE:latest" .

                            echo "Pushing versioned image..."

                            docker push "$DOCKER_IMAGE:$BUILD_NUMBER"

                            echo "Pushing latest image..."

                            docker push "$DOCKER_IMAGE:latest"

                            docker logout
                        '''
                    }
                }
            }
        }

        // 4. Update Kubernetes deployment manifest
        stage('Update Manifest Repo') {
            steps {
                dir('manifest-workspace') {

                    deleteDir()

                    withCredentials([
                        usernamePassword(
                            credentialsId: "${GIT_CREDENTIALS_ID}",
                            usernameVariable: 'GH_USER',
                            passwordVariable: 'GH_TOKEN'
                        )
                    ]) {
                        sh '''
                            set -e

                            echo "Cloning manifest repository..."

                            git clone \
                                --branch "$MANIFEST_BRANCH" \
                                "$MANIFEST_REPO" .

                            # Configure authentication for GitHub push
                            git remote set-url origin \
                                "https://${GH_USER}:${GH_TOKEN}@github.com/Johnjose10/fast-api-chatbot.git"

                            # Check manifest file
                            if [ ! -f k8s/Deployment.yaml ]; then
                                echo "ERROR: k8s/Deployment.yaml not found!"
                                exit 1
                            fi

                            echo "Updating Kubernetes image..."

                            sed -i '/name: fast-api-chatbot/{n;s|image:.*|image: '"$DOCKER_IMAGE:$BUILD_NUMBER"'|;}' \
                                k8s/Deployment.yaml

                            # Configure Git user
                            git config user.name "Johnjose10"
                            git config user.email "jhon123jose@gmail.com"

                            # Check whether the manifest changed
                            if git diff --quiet -- k8s/Deployment.yaml; then
                                echo "No manifest changes detected."
                            else
                                git add k8s/Deployment.yaml

                                git commit -m "chore(manifest): update fast-api-chatbot image tag to $BUILD_NUMBER [skip ci]"

                                echo "Pushing updated manifest to GitHub..."

                                git push origin "$MANIFEST_BRANCH"
                            fi

                            # Remove token from Git remote configuration
                            git remote set-url origin "$MANIFEST_REPO"

                            echo "Manifest repository updated successfully."
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo '''
            ==========================================
            PIPELINE EXECUTED SUCCESSFULLY!
            ==========================================
            Source code checked out.
            SonarQube analysis completed.
            Docker image built and pushed.
            Kubernetes manifest updated.
            ==========================================
            '''
        }

        failure {
            echo 'Pipeline failed. Please check the stage logs.'
        }

        always {
            echo 'Jenkins pipeline execution completed.'
        }
    }
}

        MANIFEST_REPO         = 'https://github.com/Johnjose10/fast-api-chatbot.git'
        MANIFEST_BRANCH       = 'main'
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool "${SONAR_SCANNER_TOOL}"
                    withSonarQubeEnv("${SONAR_SERVER_NAME}") {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=fast-api-chatbot-key \
                            -Dsonar.projectName='fast-api-chatbot-project' \
                            -Dsonar.sources=. \
                            -Dsonar.exclusions=*/venv/,/.venv/,/_pycache_/,/.pytest_cache/,/static/*"
                    }
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin
                            
                            docker build -t ${DOCKER_IMAGE}:${env.BUILD_NUMBER} -t${DOCKER_IMAGE}:latest .
                            
                            docker push ${DOCKER_IMAGE}:${env.BUILD_NUMBER}
                            docker push ${DOCKER_IMAGE}:latest
                        """
                    }
                }
            }
        }

        stage('Update Manifest Repo') {
            steps {
                dir('manifest-workspace') {
                    sh "git clone ${MANIFEST_REPO} . || true"
                    withCredentials([usernamePassword(credentialsId: "${GIT_CREDENTIALS_ID}", usernameVariable: 'GH_USER', passwordVariable: 'GH_TOKEN')]) {
                        sh """
                            git remote set-url origin https://\${GH_USER}:\${GH_TOKEN}@github.com/Johnjose10/fast-api-chatbot.git
                            git pull --rebase origin ${MANIFEST_BRANCH}
                            
                            sed -i '/name: fast-api-chatbot/{n;s|image:.*|image: ${DOCKER_IMAGE}:${env.BUILD_NUMBER}|}' k8s/Deployment.yaml
                            
                            git config user.name "Johnjose10"
                            git config user.email "jhon123jose@gmail.com"
                            git commit -am "chore(manifest): update fast-api-chatbot image tag to ${env.BUILD_NUMBER} [skip ci]"
                            git push origin ${MANIFEST_BRANCH}
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline executed successfully! fast-api-chatbot image built, pushed, and manifest repo updated.'
        }
        failure {
            echo 'Pipeline failed. Please review the stage logs above.'
        }
    }
}
