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
                    docker run -d --name test-${BUILD_NUMBER} webapp:${BUILD_NUMBER}
                    sleep 2
                    docker ps | grep test-${BUILD_NUMBER}
                    docker logs test-${BUILD_NUMBER}
                    docker stop test-${BUILD_NUMBER}
                    docker rm test-${BUILD_NUMBER}
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Uygulama deploy ediliyor...'
                sh '''
                    # Eski container'ları temizle
                    docker ps -a | grep webapp-prod | awk '{print $1}' | xargs -r docker rm -f || true
                    
                    # Port 9090'ı kullanan container'ı bul ve durdur
                    docker ps | grep ':9090->' | awk '{print $1}' | xargs -r docker stop || true
                    
                    # Yeni container'ı başlat
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
