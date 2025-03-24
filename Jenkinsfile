pipeline {
    agent  { label 'agent-3' }

    environment {
        // GitHub secrets passed as environment variables
        PORT_SECRET = credentials('PORT')
        DBNAME_SECRET = credentials('dbName')
        MONGO_URI_SECRET = credentials('MONGO_URI')
        VITE_API_URL = credentials('VITE_API_URL')
        // Tag based on the current build number or Git commit hash
        IMAGE_TAG = "v${BUILD_NUMBER}"  // Use Jenkins build number or other tags
        DOCKER_HUB_TOKEN = credentials('docker-pat')
        DOCKER_HUB_USER = credentials('docker-username')
    }

    stages {
        stage('Checkout Repositories') {
            steps {
                parallel(
                    frontend: {
                        echo "Cloning frontend repository"
                        dir('frontend') {  // Clone into a subdirectory named "frontend"
                            git url: 'https://github.com/Ashmit-Kumar/note-taker.git', branch: 'main', credentialsId: 'github-credential'
                        }
                    },
                    backend: {
                        echo "Cloning backend repository"
                        dir('backend') {  // Clone into a subdirectory named "backend"
                            git url: 'https://github.com/Ashmit-Kumar/note-taker-backend.git', branch: 'main', credentialsId: 'github-credential'
                        }
                    },
                )
            }
        }

        stage('Docker Setup') {
            steps {
                script {
                    // Docker login with personal access token
                    echo "Logging into Docker Hub"
                    sh """
                    docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_TOKEN}
                    """

                    // Check Docker Compose version
                    echo "Checking Docker Compose version"
                    sh "docker-compose --version"
                }
            }
        }

        stage('Build Backend') {
            steps {
                script {
                    // Export environment variables before calling docker-compose
                    echo "Building the frontend Docker image with tag ${IMAGE_TAG}"
                    sh """
                    export PORT=${PORT_SECRET}
                    export dbName=${DBNAME_SECRET}
                    export MONGO_URI=${MONGO_URI_SECRET}
                    sh "docker-compose -f /home/ubuntu/workspace/second-pipeline/backend/docker-compose.yml build --no-cache --build-arg IMAGE_TAG=${IMAGE_TAG} backend"
                }
            }
        }

        stage('Build Frontend') {
            steps {
                script {
                    // Build the frontend Docker image with a tag
                    echo "Building the frontend Docker image with tag ${IMAGE_TAG}"
                    sh "docker-compose -f /home/ubuntu/workspace/second-pipeline/frontend/docker-compose.yml build --no-cache --build-arg IMAGE_TAG=${IMAGE_TAG} frontend"
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                script {
                    // Deploy the previously built images (no need to rebuild them)
                    echo "Deploying to staging with tag ${IMAGE_TAG}"
                    sh 'docker-compose -f workspace/second-pipeline/docker-compose.yml up -d'  // Run containers in detached mode
                }
            }
        }
    }

    post {
        always {
            // This section is optional. You can add notifications or any final steps here.
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
