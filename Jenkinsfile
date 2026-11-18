
pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-cred'
        DOCKER_IMAGE          = 'manasavenu2004/fastapi-app'
        SONAR_SERVER_NAME     = 'sonarqube-env'
        SONAR_SCANNER_TOOL    = 'sonar-scanner'
        GIT_CREDENTIALS_ID    = 'github-cred'

        MANIFEST_REPO         = 'https://github.com/manasavenu-2004/Simple_FastAPI_Application.git'
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
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=fastapi-app-key \
                            -Dsonar.projectName=fastapi-app-project \
                            -Dsonar.sources=. \
                            -Dsonar.exclusions=**/manifests/**,**/.git/**
                        """
                    }
                }
            }
        }

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
                            echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                            docker build \
                            -t ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                            -t ${DOCKER_IMAGE}:latest .

                            docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                            docker push ${DOCKER_IMAGE}:latest

                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Update Manifest Repo') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${GIT_CREDENTIALS_ID}",
                        usernameVariable: 'GH_USER',
                        passwordVariable: 'GH_TOKEN'
                    )
                ]) {
                    sh '''
                        git remote set-url origin \
                        "https://${GH_USER}:${GH_TOKEN}@github.com/manasavenu-2004/Simple_FastAPI_Application.git"

                        git pull --rebase origin ${MANIFEST_BRANCH}

                        sed -i "s|image: .*|image: ${DOCKER_IMAGE}:${BUILD_NUMBER}|" manifests/Deployment.yaml

                        git config user.name "manasavenu-2004"
                        git config user.email "manasavenu88914@gmail.com"

                        git add manifests/Deployment.yaml

                        if ! git diff --cached --quiet; then
                            git commit -m "Update FastAPI image to ${BUILD_NUMBER} [skip ci]"
                            git push origin ${MANIFEST_BRANCH}
                        else
                            echo "No manifest changes to commit."
                        fi
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'FastAPI pipeline executed successfully! Image built, pushed, and manifest updated.'
        }

        failure {
            echo 'Pipeline failed. Please check the console output.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
