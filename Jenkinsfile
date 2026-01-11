pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'myrepo/webapp'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Build') {
            steps {
                echo 'Docker imaji olusturuluyor...'
                sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
                sh 'docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest'
            }
        }
        
        stage('Push') {
            steps {
                echo 'Docker imaji push ediliyor...'
                sh 'docker push ${DOCKER_IMAGE}:${DOCKER_TAG}'
                sh 'docker push ${DOCKER_IMAGE}:latest'
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Kubernetes deploy ediliyor...'
                sh 'kubectl apply -f k8s/'
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline basariyla tamamlandi!'
        }
        failure {
            echo 'Pipeline basarisiz oldu!'
        }
    }
}
