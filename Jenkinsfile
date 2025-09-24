// This Jenkinsfile is tailored for the nodejs-demo-app.
// It builds the Docker image from the Dockerfile in the repository,
// runs 'npm test', and then pushes the image to a Docker registry.

pipeline {
    // 1. Agent Configuration
    // The pipeline will run on any available Jenkins agent.
    agent any

    // 2. Environment Variables
    // Centralized configuration for the pipeline.
    environment {
        // *** IMPORTANT ***
        // Change 'your-docker-repo' to your Docker Hub username or private registry URL.
        IMAGE_NAME = 'chethankl/nodejs-demo-app'
        
        // Tags the Docker image with the current build number for versioning.
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        
        // *** IMPORTANT ***
        // This is the ID of the 'Username/Password' credential you must create in Jenkins
        // for your Docker Hub account.
        DOCKER_CREDENTIALS_ID = 'your-dockerhub-credentials'
    }

    // 3. Pipeline Stages
    // The CI/CD process is broken down into logical stages.
    stages {

        // Stage 1: Build the Docker Image
        // This stage uses the Dockerfile from your Git repository to build the application image.
        stage('Build') {
            steps {
                script {
                    echo "Building the Docker image: ${IMAGE_NAME}:${IMAGE_TAG}"
                    // The '.' indicates that the Dockerfile is in the root of the workspace.
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}", '.')
                }
            }
        }

        // Stage 2: Run Tests
        // This stage runs the test script defined in your package.json inside the new Docker image.
        stage('Test') {
            steps {
                script {
                    echo "Running tests inside the container..."
                    // NOTE: Your current 'package.json' test script is: "echo \"Error: no test specified\" && exit 1"
                    // This will cause the pipeline to fail, which is the correct behavior for a failing test.
                    // To make this stage pass, update the 'test' script in your package.json to run real tests
                    // or change it to something like: "echo \"Tests passed\" && exit 0"
                    docker.image("${IMAGE_NAME}:${IMAGE_TAG}").inside {
                        sh 'npm test'
                    }
                }
            }
        }

        // Stage 3: Deploy the Docker Image
        // On success, this stage logs into Docker Hub and pushes the image.
        stage('Deploy') {
            steps {
                script {
                    echo "Deploying the Docker image to registry..."
                    // Logs into the registry using the credentials stored in Jenkins.
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        
                        // Push the build-specific tag.
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()

                        // Also push the 'latest' tag for easy access to the most recent version.
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push('latest')
                    }
                }
            }
        }
    }

    // 4. Post-Build Actions
    // These actions are executed after all stages are complete.
    post {
        // 'always' runs regardless of the pipeline's success or failure.
        always {
            echo 'Pipeline finished.'
            // Good practice to clean up the built image from the Jenkins agent to save disk space.
            // '|| true' ensures this step doesn't fail the build if the image is not found.
            sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
        }
        // 'success' runs only if the pipeline was successful.
        success {
            echo 'Pipeline deployed successfully!'
        }
        // 'failure' runs only if the pipeline failed.
        failure {
            echo 'Pipeline failed. Please review the logs.'
        }
    }
}

