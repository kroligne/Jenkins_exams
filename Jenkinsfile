pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/kroligne/Jenkins_exams.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker build -t $DOCKER_USERNAME/movie_service:latest ./movie-service'
                sh 'docker build -t $DOCKER_USERNAME/cast_service:latest ./cast-service'
                sh 'docker build -t $DOCKER_USERNAME/nginx:latest .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) {
                    sh 'docker push $DOCKER_USERNAME/movie_service:latest'
                    sh 'docker push $DOCKER_USERNAME/cast_service:latest'
                    sh 'docker push $DOCKER_USERNAME/nginx:latest'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                branch 'main'
            }
            steps {
                sh 'kubectl apply -f charts/templates/deployment.yaml'
                sh 'kubectl apply -f charts/templates/service.yaml'
            }
        }
    }
}
