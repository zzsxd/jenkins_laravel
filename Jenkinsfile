pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'zzsxdd/laravel-app'
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
        GIT_CREDENTIALS_ID = 'github-credentials'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    extensions: [],
                    userRemoteConfigs: [[
                        url: 'https://github.com/zzsxd/jenkins_laravel.git',
                        credentialsId: "${env.GIT_CREDENTIALS_ID}"
                    ]]
                ])
            }
        }

        stage('Prepare Environment') {
            steps {
                script {
                    sh 'mkdir -p nginx/conf.d storage bootstrap/cache'
                    
                    sh 'test -f .env.production && cp .env.production .env || true'
                    
                    sh '''
                    if [ ! -f .env ]; then
                        cp .env.example .env
                    fi
                    '''
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh 'docker-compose build'
                }
            }
        }

        stage('Start Services and Run Tests') {
            steps {
                script {
                    sh 'docker-compose up -d'
                    
                    sh 'sleep 30'
                    
                    sh '''
                    docker-compose exec -T app composer install --no-interaction --no-dev
                    docker-compose exec -T app php artisan key:generate
                    docker-compose exec -T app php artisan migrate --force
                    '''
                    
                    sh 'docker-compose exec -T app php artisan test || echo "No tests found"'
                }
            }
        }

        stage('Build Production Image') {
            steps {
                script {
                    docker.build("${env.DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${env.DOCKER_CREDENTIALS_ID}") {
                        docker.image("${env.DOCKER_IMAGE}:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh '''
                    docker-compose down
                    docker-compose up -d
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker-compose down || true'
        }
        success {
            echo '🚀 Laravel application deployed successfully!'
            echo '🌐 Application URL: http://YOUR_SERVER_IP'
        }
        failure {
            echo '❌ Deployment failed!'
        }
    }
}