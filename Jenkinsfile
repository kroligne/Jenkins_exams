pipeline {
    agent any

    environment {
        DOCKER_USER    = "kroligne" 
        DOCKER_REGISTRY = "docker.io"
        MOVIE_IMAGE    = "movie-service"
        CAST_IMAGE     = "cast-service"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/kroligne/Jenkins_exams.git'
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build movie-service') {
                    steps {
                        dir('movie-service') {
                            sh """
                                docker build -t \$DOCKER_REGISTRY/\$DOCKER_USER/\$MOVIE_IMAGE:\${BUILD_NUMBER} .
                            """
                        }
                    }
                }
                stage('Build cast-service') {
                    steps {
                        dir('cast-service') {
                            sh """
                                docker build -t \$DOCKER_REGISTRY/\$DOCKER_USER/\$CAST_IMAGE:\${BUILD_NUMBER} .
                            """
                        }
                    }
                }
            }
        }

        stage('Push Docker Images') {
            parallel {
                stage('Push movie-service') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', 
                                                          usernameVariable: 'USERNAME', 
                                                          passwordVariable: 'PASSWORD')]) {
                            sh """
                                echo \$PASSWORD | docker login -u \$USERNAME --password-stdin \$DOCKER_REGISTRY
                                docker push \$DOCKER_REGISTRY/\$DOCKER_USER/\$MOVIE_IMAGE:\${BUILD_NUMBER}
                            """
                        }
                    }
                }
                stage('Push cast-service') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', 
                                                          usernameVariable: 'USERNAME', 
                                                          passwordVariable: 'PASSWORD')]) {
                            sh """
                                echo \$PASSWORD | docker login -u \$USERNAME --password-stdin \$DOCKER_REGISTRY
                                docker push \$DOCKER_REGISTRY/\$DOCKER_USER/\$CAST_IMAGE:\${BUILD_NUMBER}
                            """
                        }
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                       helm upgrade --install jenkins-exam ./charts \\
                         --set movieService.image.repository=\$DOCKER_REGISTRY/\$DOCKER_USER/\$MOVIE_IMAGE \\
                         --set movieService.image.tag=\${BUILD_NUMBER} \\
                         --set castService.image.repository=\$DOCKER_REGISTRY/\$DOCKER_USER/\$CAST_IMAGE \\
                         --set castService.image.tag=\${BUILD_NUMBER}
                    """
                }
            }
        }
    }
}
