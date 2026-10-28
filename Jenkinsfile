
pipeline {
    agent any

    environment {
        IMAGE_NAME = "phanindra61/board-game"
        TAG = "${BUILD_NUMBER}"
        FULL_IMAGE = "${IMAGE_NAME}:${TAG}"
    }

    stages {

        // 1. Checkout source code from GitHub
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/phanindra61/BoardGame.git'
            }
        }

        // 2. Docker Hub Login
        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'DOCKERHUB_ID',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        set +x
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" \
                            --password-stdin
                    '''
                }
            }
        }

        // 3. Build Java Application
        stage('Building application') {
            steps {
                sh 'mvn clean package'
            }
        }

        // 4. Build Docker Image
        stage('Building docker image') {
            steps {
                sh 'docker build -t "$FULL_IMAGE" .'
            }
        }

        // 5. Push Docker Image to Docker Hub
        stage('Push to DockerHub') {
            steps {
                sh 'docker push "$FULL_IMAGE"'
            }
        }

        // 6. Update Kubernetes Manifest (GitOps)
        stage('Update Git (GitOps)') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'GITHUB_CRED',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_PASS'
                    )
                ]) {
                    sh '''
                        set +x

                        git config user.email "apoorvaramesh11@gmail.com"
                        git config user.name "apoorvaramesh11"

                        sed -i "s|image:.*|image: ${FULL_IMAGE}|" \
                            manifests/deployment-service.yaml

                        git add manifests/deployment-service.yaml

                        if ! git diff --cached --quiet; then
                            git commit -m "Update image ${FULL_IMAGE}"
                        else
                            echo "No changes to commit"
                        fi

                        AUTH=$(printf '%s:%s' "$GIT_USER" "$GIT_PASS" \
                            | base64 | tr -d '\\n')

                        git -c http.extraheader="AUTHORIZATION: basic $AUTH" \
                            push origin HEAD:main

                        unset AUTH
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Jenkins console output.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
