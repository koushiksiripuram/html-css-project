pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "koushiksiripuram/html-argocd"
        GITOPS_REPO = "https://github.com/koushiksiripuram/html-argocd-css.git"
        GITOPS_BRANCH = "main"
        MANIFEST_PATH = "dev/deployment.yaml"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    IMAGE_TAG = "${BUILD_NUMBER}.0"
                    sh "docker build -t $DOCKER_IMAGE:$IMAGE_TAG ."
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                 usernameVariable: 'DOCKER_USER',
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push $DOCKER_IMAGE:$IMAGE_TAG"
                }
            }
        }

        stage('Update GitOps Repo') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-creds',
                                                 usernameVariable: 'GIT_USER',
                                                 passwordVariable: 'GIT_PASS')]) {
                    sh """
                    rm -rf gitops
                    git clone https://$GIT_USER:$GIT_PASS@${GITOPS_REPO.replace('https://','')} gitops
                    cd gitops
                    sed -i 's|image: .*|image: $DOCKER_IMAGE:$IMAGE_TAG|' $MANIFEST_PATH
                    git config user.email "koushiksiripuram48@gmail.com"
                    git config user.name "koushiksiripuram"
                    git add .
                    git commit -m "Update image to $IMAGE_TAG"
                    git push origin $GITOPS_BRANCH
                    """
                }
            }
        }
    }
}
