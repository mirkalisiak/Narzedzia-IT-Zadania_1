pipeline {
    agent any
    stages {
        stage('Clone') {
            steps {
                git '[https://github.com/Matiorus/Django-build.git](https://github.com/Matiorus/Django-build.git)'
            }
        }
        stage('List Files') {
            steps {
                sh 'ls -la'
            }
        }
        stage('Build Django in Docker') {
            steps {
                // Budowanie obrazu na podstawie Dockerfile z repozytorium
                sh 'docker build -t django-app .'
                
                // Zapis obrazu do pliku tar w celu archiwizacji
                sh 'docker save django-app > django-app.tar'
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'django-app.tar', followSymlinks: false
        }
    }
}
