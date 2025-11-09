pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
        PROJECT_NAME = 'ecommerce-jenkins'
    }

    stages {
        stage('Cleanup') {
            steps {
                script {
                    echo 'Cleaning up previous containers...'
                    sh '''
                        docker-compose down -v || true
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo 'Starting containerized application...'
                    sh '''
                        docker-compose up -d --build
                    '''
                }
            }
        }

        stage('Verify') {
            steps {
                script {
                    echo 'Verifying containers are running...'
                    sh '''
                        docker ps
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
            echo 'Frontend: http://your-ec2-ip:3001'
        }
        failure {
            echo 'Pipeline failed!'
            sh 'docker-compose logs || true'
        }
        always {
            echo 'Cleaning up workspace...'
        }
    }
}