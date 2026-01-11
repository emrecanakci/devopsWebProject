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
        
        stage('Test') {
            steps {
                echo 'Container test ediliyor...'
                sh '''
                    docker run -d --name test-${BUILD_NUMBER} -p 8888:80 ${DOCKER_IMAGE}:${DOCKER_TAG}
                    sleep 3
                    curl -f http://localhost:8888 || exit 1
                    docker stop test-${BUILD_NUMBER}
                    docker rm test-${BUILD_NUMBER}
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Uygulama deploy ediliyor...'
                sh '''
                    docker stop webapp-prod || true
                    docker rm webapp-prod || true
                    docker run -d --name webapp-prod -p 9090:80 ${DOCKER_IMAGE}:latest
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline basariyla tamamlandi!'
            echo 'Uygulama http://localhost:9090 adresinde calisıyor'
        }
        failure {
            echo 'Pipeline basarisiz oldu!'
        }
        always {
            sh 'docker system prune -f || true'
        }
    }
}
