pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'zzsxdd/laravel-app'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/zzsxd/jenkins_laravel.git'
            }
        }

        stage('Prepare') {
            steps {
                sh 'mkdir -p nginx/conf.d storage bootstrap/cache'
                sh 'cp .env.example .env'
            }
        }

        stage('Build and Test') {
            steps {
                script {
                    sh 'docker compose build'
                    sh 'docker compose up -d'
                    sh 'sleep 30'
                    
                    sh '''
                    docker compose exec -T app composer install --no-interaction --no-dev
                    docker compose exec -T app php artisan key:generate
                    docker compose exec -T app php artisan migrate --force
                    docker compose exec -T app php artisan route:list || echo "No routes command"
                    '''
                    
                    echo '✅ Application setup completed successfully!'
                }
            }
        }

        stage('Build Production Image') {
            steps {
                script {
                    sh "docker build -t ${env.DOCKER_IMAGE}:${env.BUILD_NUMBER} ."
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo '🚀 Application would be deployed here'
                    echo '🌐 Access at: http://localhost'
                }
            }
        }
    }

    post {
        always {
            sh 'docker compose down || true'
        }
        success {
            echo '🎉 CI/CD Pipeline completed successfully!'
            echo '✅ Docker images built'
            echo '✅ Dependencies installed'
            echo '✅ Database migrated'
            echo '✅ Application ready'
        }
    }
}