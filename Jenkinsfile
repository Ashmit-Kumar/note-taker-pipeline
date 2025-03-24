pipeline {
    agent any

    environment {
        // GitHub secrets passed as environment variables
        PORT_SECRET = credentials('PORT')
        DBNAME_SECRET = credentials('dbName')
        MONGO_URI_SECRET = credentials('MONGO_URI')
        VITE_API_URL = '13.203.210.224' // Directly set EC2 IP
        IMAGE_TAG = "v${BUILD_NUMBER}"
        DOCKER_HUB_TOKEN = credentials('docker-pat')
        DOCKER_HUB_USER = credentials('docker-username')
    }

    stages {
        stage('Checkout Repositories') {
            steps {
                parallel(
                    frontend: {
                        echo "Cloning frontend repository"
                        git url: 'https://github.com/Ashmit-Kumar/note-taker.git', branch: 'main', credentialsId: 'github-credential'
                    },
                    backend: {
                        echo "Cloning backend repository"
                        git url: 'https://github.com/Ashmit-Kumar/note-taker-backend.git', branch: 'main', credentialsId: 'github-credential'
                    },
                    pipelineRepo: {
                        echo "Cloning pipeline repository"
                        git url: 'https://github.com/Ashmit-Kumar/note-taker-pipeline.git', branch: 'main', credentialsId: 'github-credential'
                    }
                )
            }
        }

        stage('Docker Setup') {
            steps {
                script {
                    echo "Logging into Docker Hub"
                    sh "docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_TOKEN}"

                    echo "Checking Docker Compose version"
                    sh "docker-compose --version"
                }
            }
        }

        stage('Build Images') {
            steps {
                parallel(
                    backendBuild: {
                        echo "Building backend Docker image"
                        sh "docker-compose -f note-taker-backend/docker-compose.yml build --no-cache --build-arg IMAGE_TAG=${IMAGE_TAG} backend"
                    },
                    frontendBuild: {
                        echo "Building frontend Docker image"
                        sh "docker-compose -f note-taker/docker-compose.yml build --no-cache --build-arg IMAGE_TAG=${IMAGE_TAG} frontend"
                    }
                )
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    echo "Running tests"
                    sh 'docker-compose -f note-taker-pipeline/docker-compose.yml run --rm backend npm test'
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    echo "Deploying to EC2 with tag ${IMAGE_TAG}"
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@${VITE_API_URL} << 'EOF'
                        docker-compose -f /path/to/docker-compose.yml pull
                        docker-compose -f /path/to/docker-compose.yml up -d
                        docker image prune -f
                    EOF
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed"
        }
        success {
            echo "Pipeline completed successfully"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
