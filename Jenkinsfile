pipeline {
    agent any

    environment {
        // GitHub secrets passed as environment variables
        PORT_SECRET = credentials('PORT')
        DBNAME_SECRET = credentials('DBNAME')
        MONGO_URI_SECRET = credentials('MONGO_URI')

        // Tag based on the current build number or Git commit hash
        IMAGE_TAG = "latest-${BUILD_NUMBER}"  // Use Jenkins build number or other tags
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

        stage('Build Frontend') {
            steps {
                script {
                    // Build the frontend Docker image with a tag
                    echo "Building the frontend Docker image with tag ${IMAGE_TAG}"
                    sh "docker-compose -f docker-compose-repo/docker-compose.yml build --no-cache --build-arg IMAGE_TAG=${IMAGE_TAG} frontend"
                }
            }
        }

        stage('Build Backend') {
            steps {
                script {
                    // Build the backend Docker image with a tag
                    echo "Building the backend Docker image with tag ${IMAGE_TAG}"
                    sh "docker-compose -f docker-compose-repo/docker-compose.yml build --no-cache --build-arg IMAGE_TAG=${IMAGE_TAG} backend"
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    // Run tests in the backend container (if applicable)
                    echo "Running tests"
                    sh 'docker-compose -f docker-compose-repo/docker-compose.yml run --rm backend npm test'  // Adjust command as needed
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

        stage('Clean Up') {
            steps {
                script {
                    // Clean up the resources after deployment (optional)
                    echo "Cleaning up"
                    sh 'docker-compose -f docker-compose-repo/docker-compose.yml down'  // Shut down the containers
                }
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
