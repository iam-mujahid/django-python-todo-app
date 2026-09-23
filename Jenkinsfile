pipeline {

    agent any

    environment {
        IMAGE_NAME = "mujahidhub/django-python-todo-app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Application') {
            steps {
                git(
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/iam-mujahid/django-python-todo-app.git',
                    branch: 'main'
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image..."
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push Docker Image to Registry') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "Logging in to Docker Hub..."
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

                        echo "Pushing Docker image..."
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}

                        docker logout
                    '''
                }
            }
        }

        stage('Checkout GitOps Repository') {
            steps {
                dir('gitops') {
                    git(
                        credentialsId: 'github-credentials',
                        url: 'https://github.com/iam-mujahid/django-python-todo-deploy-manifests.git',
                        branch: 'main'
                    )
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            steps {
                dir('gitops') {
                    sh '''
                        echo "Current deployment image:"
                        grep "image:" deploy.yaml

                        echo "Updating image to:"
                        echo "${IMAGE_NAME}:${IMAGE_TAG}"

                        sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" deploy.yaml

                        echo "Updated deployment image:"
                        grep "image:" deploy.yaml
                    '''
                }
            }
        }

        stage('Commit and Push GitOps Changes') {
            steps {
                dir('gitops') {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'github-credentials',
                            usernameVariable: 'GIT_USERNAME',
                            passwordVariable: 'GIT_PASSWORD'
                        )
                    ]) {
                        sh '''
                            git config user.name "Jenkins"
                            git config user.email "jenkins@localhost"

                            git add deploy.yaml

                            git commit -m "Update Django Todo image to ${IMAGE_TAG}" || echo "No changes to commit"

                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/iam-mujahid/django-python-todo-deploy-manifests.git HEAD:main
                        '''
                    }
                }
            }
        }
    }
}
