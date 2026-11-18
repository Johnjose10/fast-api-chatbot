
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

        // Stage 1: Checkout source code
        stage('Checkout Source Code') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        // Stage 2: SonarQube analysis
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool "${SONAR_SCANNER_TOOL}"

                    withSonarQubeEnv("${SONAR_SERVER_NAME}") {
                        sh """
                            "${scannerHome}/bin/sonar-scanner" \
                            -Dsonar.projectKey=fast-api-chatbot-key \
                            -Dsonar.projectName=fast-api-chatbot-project \
                            -Dsonar.sources=. \
                            -Dsonar.exclusions=**/venv/**,**/.venv/**,**/__pycache__/**,**/.pytest_cache/**,**/static/**
                        """
                    }
                }
            }
        }

        // Stage 3: Build and push Docker image
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

        // Stage 4: Update Kubernetes manifest in GitHub
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

                            echo "Configuring GitHub authentication..."

                            git remote set-url origin \
                                "https://${GH_USER}:${GH_TOKEN}@github.com/Johnjose10/fast-api-chatbot.git"

                            if [ ! -f k8s/Deployment.yaml ]; then
                                echo "ERROR: k8s/Deployment.yaml not found!"
                                exit 1
                            fi

                            echo "Updating Docker image in Deployment.yaml..."

                            sed -i '/name: fast-api-chatbot/{n;s|image:.*|image: '"$DOCKER_IMAGE:$BUILD_NUMBER"'|;}' \
                                k8s/Deployment.yaml

                            git config user.name "Johnjose10"
                            git config user.email "jhon123jose@gmail.com"

                            if git diff --quiet -- k8s/Deployment.yaml; then
                                echo "No changes detected in Deployment.yaml."
                            else
                                git add k8s/Deployment.yaml

                                git commit -m "Update fast-api-chatbot image to build $BUILD_NUMBER [skip ci]"

                                echo "Pushing updated manifest..."

                                git push origin "$MANIFEST_BRANCH"
                            fi

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
            echo 'SUCCESS: Source checked out, SonarQube analysis completed, Docker image pushed, and Kubernetes manifest updated.'
        }

        failure {
            echo 'FAILURE: Pipeline failed. Check the console output for details.'
        }

        always {
            echo 'Jenkins pipeline execution completed.'
        }
    }
}
