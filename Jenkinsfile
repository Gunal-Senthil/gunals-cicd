pipeline {
    agent any

    environment {
        IMAGE_NAME = "gunalsenthil/gunals:v${BUILD_NUMBER}"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $IMAGE_NAME
                    docker logout
                    '''
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            steps {
                sh '''
                sed -i "s|image:.*|image: gunalsenthil/gunals:v${BUILD_NUMBER}|g" k8s/deployment.yaml
                '''
            }
        }

        stage('Commit and Push Manifest') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github',
                    usernameVariable: 'GITHUB_USER',
                    passwordVariable: 'GITHUB_TOKEN'
                )]) {

                    sh '''
                    git config user.name "Jenkins"
                    git config user.email "jenkins@example.com"

                    git add k8s/deployment.yaml

                    git commit -m "Update image to v${BUILD_NUMBER}" || true

                    git push https://${GITHUB_USER}:${GITHUB_TOKEN}@github.com/Gunal-Senthil/gunals-cicd.git HEAD:main
                    '''
                }
            }
        }
    }
}
