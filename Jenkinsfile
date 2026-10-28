pipeline {
    agent any

    environment {
        IMAGE_NAME = "apoorvar12/board-game"
        TAG = "${BUILD_NUMBER}"
        FULL_IMAGE = "${IMAGE_NAME}:${TAG}"
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/phanindra61/BoardGame.git'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'DOCKERHUB_ID',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Building application') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Building docker image') {
            steps {
                sh "docker build -t ${FULL_IMAGE} ."
            }
        }

        stage('Push to dockerhub') {
            steps {
                sh "docker push ${FULL_IMAGE}"
            }
        }

        stage('Update Git (GitOps)') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'GITHUB_CRED',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {

                    sh """
                    git config user.email "apoorvaramesh11@gmail.com"
                    git config user.name "apoorvaramesh11"

                    sed -i "s|image:.*|image: ${FULL_IMAGE}|" manifests/deployment-service.yaml

                    git add manifests/deployment-service.yaml
                    git commit -m "Update image ${FULL_IMAGE}" || echo "No changes"

                    git push https://${GIT_USER}:${GIT_PASS}@github.com/phanindra61/BoardGame.git HEAD:main
                    """
                }
            }
        }
    }
}
