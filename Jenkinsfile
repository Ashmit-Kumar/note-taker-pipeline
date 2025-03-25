pipeline {
    agent { label 'agent-3' }

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
                    }
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
                withEnv([
                    "PORT=${PORT_SECRET}",
                    "dbName=${DBNAME_SECRET}",
                    "MONGO_URI=${MONGO_URI_SECRET}"
                ]) {
                    script {
                        // Create .env file for backend
                        echo "Creating .env file for the backend"
                        sh """
                        echo "PORT=${PORT_SECRET}" > /home/ubuntu/workspace/second-pipeline/backend/.env
                        echo "DBNAME=${DBNAME_SECRET}" >> /home/ubuntu/workspace/second-pipeline/backend/.env
                        echo "MONGO_URI=${MONGO_URI_SECRET}" >> /home/ubuntu/workspace/second-pipeline/backend/.env
                        """
                        
                        // Build the backend Docker image with tag ${IMAGE_TAG}
                        echo "Building the backend Docker image with tag ${IMAGE_TAG}"
                        sh """
                        sudo docker-compose -f /home/ubuntu/workspace/second-pipeline/backend/docker-compose.yml build --no-cache --build-arg IMAGE_TAG=${IMAGE_TAG} backend
                        """
                    }
                }
            }
        }

        stage('Build Frontend') {
            steps {
                script {
                    // Create .env file for frontend
                    echo "Creating .env file for the frontend"
                    sh """
                    echo "VITE_API_URL=${VITE_API_URL}" > /home/ubuntu/workspace/second-pipeline/frontend/.env
                    """
                    
                    // Build the frontend Docker image with a tag
                    echo "Building the frontend Docker image with tag ${IMAGE_TAG}"
                    sh """
                    sudo docker-compose -f /home/ubuntu/workspace/second-pipeline/frontend/docker-compose.yml build --no-cache --build-arg IMAGE_TAG=${IMAGE_TAG} frontend
                    """
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                script {
                    // Create .env file for both frontend and backend in the deploy stage
                    echo "Creating .env file for deployment"
                    sh """
                    echo "PORT=${PORT_SECRET}" > /home/ubuntu/workspace/second-pipeline/.env
                    echo "DBNAME=${DBNAME_SECRET}" >> /home/ubuntu/workspace/second-pipeline/.env
                    echo "MONGO_URI=${MONGO_URI_SECRET}" >> /home/ubuntu/workspace/second-pipeline/.env
                    echo "VITE_API_URL=${VITE_API_URL}" >> /home/ubuntu/workspace/second-pipeline/.env
                    """
                    
                    // Deploy the previously built images (no need to rebuild them)
                    echo "Deploying to staging with tag ${IMAGE_TAG}"
                    sh """	
                    sudo docker-compose -f /home/ubuntu/workspace/second-pipeline/docker-compose.yml up -d
                    """
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
