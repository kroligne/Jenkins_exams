pipeline {
    agent any
    
    environment {
        DOCKER_USER    = "kroligne" 
        DOCKER_IMAGE   = "jenkins-exam"
        DOCKER_REGISTRY = "docker.io"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/kroligne/Jenkins_exams.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                       docker build -t \$DOCKER_REGISTRY/\$DOCKER_USER/\$DOCKER_IMAGE:\${BUILD_NUMBER} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', 
                                                      usernameVariable: 'USERNAME', 
                                                      passwordVariable: 'PASSWORD')]) {
                        sh """
                           echo \$PASSWORD | docker login -u \$USERNAME --password-stdin \$DOCKER_REGISTRY
                           docker push \$DOCKER_REGISTRY/\$DOCKER_USER/\$DOCKER_IMAGE:\${BUILD_NUMBER}
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                       helm upgrade --install jenkins-exam ./charts \
                         --set image.repository=\$DOCKER_REGISTRY/\$DOCKER_USER/\$DOCKER_IMAGE \
                         --set image.tag=\${BUILD_NUMBER}
                    """
                }
            }
        }
    }
}

