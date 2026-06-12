pipeline {
    agent any

    environment {
        // Your GitHub username or organization name, used to build the image URL
        GITHUB_OWNER = 'piyush5621'
        // FRONTEND_URL configures CORS on the backend. Replace this with your new EC2 IP.
        FRONTEND_URL = 'http://3.27.170.194'
    }

    stages {
        stage('Validate Environment') {
            steps {
                echo "Validating Docker and System Environment..."
                sh 'docker --version'
                sh 'docker compose version'
            }
        }

        stage('Pull Images') {
            steps {
                echo "Pulling latest Docker images from GHCR..."
                sh '''
                    export GITHUB_REPOSITORY_OWNER=${GITHUB_OWNER}
                    docker compose -f docker-compose.prod.yml pull
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                withCredentials([file(credentialsId: 'backend-env-file', variable: 'BACKEND_ENV')]) {
                    sh '''
                        echo "Deploying to EC2 via Docker Compose..."
                        
                        # Clean up any leftover .env from failed builds, then securely copy the new one
                        rm -f .env
                        cp $BACKEND_ENV .env
                        
                        # Export variables for docker-compose.prod.yml
                        export GITHUB_REPOSITORY_OWNER=${GITHUB_OWNER}
                        export FRONTEND_URL=${FRONTEND_URL}
                        
                        # Restart the containers with the new images
                        docker compose -f docker-compose.prod.yml up -d
                        
                        # Clean up the .env file from the filesystem for security
                        rm .env
                        
                        echo "Containers started!"
                    '''
                }
            }
        }

        stage('Health Check') {
            steps {
                echo "Verifying application health..."
                // Sleep for 3 seconds to let backend start listening
                sleep 3
                sh 'curl --fail http://localhost:3000/health-check'
                echo "Deployment successfully verified!"
            }
        }
    }
}
