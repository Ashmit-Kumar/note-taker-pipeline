pipeline {
    agent any

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
                script {
                    // Clone frontend repository
                    echo "Cloning frontend repository"
                    git url: 'https://github.com/Ashmit-Kumar/note-taker.git', branch: 'main', credentialsId: 'your-github-credential'
                    
                    // Clone backend repository
                    echo "Cloning backend repository"
                    git url: 'https://github.com/Ashmit-Kumar/note-taker-backend.git', branch: 'main', credentialsId: 'your-github-credential'
                    
                    // Clone docker-compose repository
                    echo "Cloning docker-compose repository"
                    git url: 'https://github.com/Ashmit-Kumar/note-taker-pipeline.git', branch: 'main', credentialsId: 'your-github-credential'
                }
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
                    // Build the backend Docker image with a tag
                    echo "Building the backend Docker image with tag ${IMAGE_TAG}"
                    sh "docker-compose -f note-taker-backend/docker-compose.yml build --no-cache --build-arg IMAGE_TAG=${IMAGE_TAG} backend"
                }
            }
        }

        stage('Build Frontend') {
            steps {
                script {
                    // Build the frontend Docker image with a tag
                    echo "Building the frontend Docker image with tag ${IMAGE_TAG}"
                    sh "docker-compose -f note-taker/docker-compose.yml build --no-cache --build-arg IMAGE_TAG=${IMAGE_TAG} frontend"
                }
            }
        }


        stage('Run Tests') {
            steps {
                script {
                    // Run tests in the container (if applicable)
                    echo "Running tests"
                    sh 'docker-compose -f note-taker-pipeline/docker-compose.yml run --rm backend npm test'  // Adjust command as needed
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                script {
                    // Deploy the previously built images (no need to rebuild them)
                    echo "Deploying to staging with tag ${IMAGE_TAG}"
                    sh 'docker-compose -f docker-compose-repo/docker-compose.yml up -d'  // Run containers in detached mode
                }
            }
        }

        

    post {
        always {
            // Clean up any resources that were used during the pipeline
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
