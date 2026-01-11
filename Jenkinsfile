pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                echo 'Docker imaji olusturuluyor...'
                sh 'docker build -t webapp:${BUILD_NUMBER} .'
                sh 'docker tag webapp:${BUILD_NUMBER} webapp:latest'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Container test ediliyor...'
                sh '''
                    docker run -d --name test-${BUILD_NUMBER} -p 8888:80 webapp:${BUILD_NUMBER}
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
                    docker run -d --name webapp-prod -p 9090:80 webapp:latest
                '''
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline basariyla tamamlandi!'
            echo '🚀 Uygulama http://localhost:9090 adresinde calisıyor'
        }
        failure {
            echo '❌ Pipeline basarisiz oldu!'
        }
    }
}
