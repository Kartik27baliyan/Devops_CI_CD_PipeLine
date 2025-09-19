pipeline {
    agent any
    
    environment {
        DOCKER_BUILDKIT = "0"
    }
    
    stages {
        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Clean up any existing image with the same name
                    sh "docker rmi kartik-python-app:${env.BUILD_ID} 2>/dev/null || true"
                    
                    // Build the Docker image
                    dockerImage = docker.build("kartik-python-app:${env.BUILD_ID}")
                }
            }
        }

        stage('Test Docker Image') {
            steps {
                script {
                    // Clean up any existing container first
                    sh "docker stop test-container-${env.BUILD_ID} 2>/dev/null || true"
                    sh "docker rm test-container-${env.BUILD_ID} 2>/dev/null || true"
                    
                    // Run container with explicit port mapping
                    try {
                        dockerContainer = dockerImage.run(
                            "--name test-container-${env.BUILD_ID} " +
                            "-p 5000:5000 " +
                            "-d"
                        )
                        
                        // Wait for application to start
                        waitForContainerStart()
                        
                        // Test the application
                        testApplication()
                        
                    } catch (Exception e) {
                        echo "Error during test stage: ${e.getMessage()}"
                        currentBuild.result = 'FAILURE'
                        error("Test stage failed: ${e.getMessage()}")
                    }
                }
            }
            post {
                always {
                    script {
                        // Ensure cleanup happens even if previous steps failed
                        cleanupContainer("test-container-${env.BUILD_ID}")
                    }
                }
            }
        }
    }
}

def waitForContainerStart() {
    // Wait with retries for the application to start
    def maxAttempts = 12
    def waitTime = 5
    
    for (int i = 0; i < maxAttempts; i++) {
        try {
            // Check if container is running
            def containerStatus = sh(
                script: "docker inspect -f '{{.State.Status}}' test-container-${env.BUILD_ID}",
                returnStdout: true
            ).trim()
            
            if (containerStatus != "running") {
                throw new Exception("Container not running")
            }
            
            // Check if application is responding
            def responseCode = sh(
                script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:5000 || echo '000'",
                returnStdout: true
            ).trim()
            
            if (responseCode == "200") {
                echo "Application started successfully after ${i * waitTime} seconds"
                return
            }
            
            echo "Application not ready yet (HTTP: ${responseCode}), waiting ${waitTime} seconds... (Attempt ${i+1}/${maxAttempts})"
            sleep(waitTime)
            
        } catch (Exception e) {
            echo "Waiting for container to start... (Attempt ${i+1}/${maxAttempts})"
            sleep(waitTime)
        }
    }
    error("Application failed to start within ${maxAttempts * waitTime} seconds")
}

def testApplication() {
    // Test if the application is running correctly
    sh '''
        # Try multiple times in case the application is still starting up
        for i in {1..5}; do
            response=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:5000) || true
            
            if [ "$response" = "200" ]; then
                content=$(curl -s http://localhost:5000)
                if echo "$content" | grep -q "Hello, DevOps World!"; then
                    echo "--- Test Passed! Application is running correctly. ---"
                    exit 0
                else
                    echo "--- Test Failed! Application returned unexpected content. ---"
                    echo "Response content: $content"
                    sleep 2
                fi
            else
                echo "--- Attempt $i: Application returned HTTP code: $response ---"
                sleep 2
            fi
        done
        echo "--- All test attempts failed ---"
        exit 1
    '''
}

def cleanupContainer(containerName) {
    // Clean up containers with better error handling
    try {
        sh """
            docker stop ${containerName} 2>/dev/null || true
            docker rm ${containerName} 2>/dev/null || true
        """
        echo "Successfully cleaned up container: ${containerName}"
    } catch (Exception e) {
        echo "Warning: Failed to clean up container ${containerName}: ${e.getMessage()}"
    }
    
    // Also clean up any dangling containers
    try {
        sh """
            docker ps -aq -f name=test-container | xargs -r docker stop 2>/dev/null || true
            docker ps -aq -f name=test-container | xargs -r docker rm 2>/dev/null || true
        """
    } catch (Exception e) {
        echo "Warning during general cleanup: ${e.getMessage()}"
    }
}
