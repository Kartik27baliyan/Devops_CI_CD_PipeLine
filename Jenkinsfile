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
                    // Build the Docker image with proper error handling
                    dockerImage = docker.build("kartik-python-app:${env.BUILD_ID}")
                }
            }
        }

        stage('Test Docker Image') {
            steps {
                script {
                    // Run container with explicit error handling
                    try {
                        dockerContainer = dockerImage.run("-p 5000:5000 -d --name test-container-${env.BUILD_ID}")
                        
                        // Wait for application to start with better approach
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

// Define functions outside pipeline to avoid CPS transformation issues
def waitForContainerStart() {
    // Wait with retries for the application to start
    def maxAttempts = 10
    def waitTime = 3
    
    for (int i = 0; i < maxAttempts; i++) {
        try {
            // Use docker exec to check if the application is ready
            sh "docker exec test-container-${env.BUILD_ID} curl -s http://localhost:5000 > /dev/null 2>&1 && echo 'Application is ready'"
            echo "Application started successfully after ${i * waitTime} seconds"
            return
        } catch (Exception e) {
            echo "Application not ready yet, waiting ${waitTime} seconds... (Attempt ${i+1}/${maxAttempts})"
            sleep(waitTime)
        }
    }
    error("Application failed to start within ${maxAttempts * waitTime} seconds")
}

def testApplication() {
    // Test if the application is running correctly
    sh '''
        response=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:5000) || true
        
        if [ "$response" = "200" ]; then
            content=$(curl -s http://localhost:5000)
            if echo "$content" | grep -q "Hello, DevOps World!"; then
                echo "--- Test Passed! Application is running correctly. ---"
            else
                echo "--- Test Failed! Application returned unexpected content. ---"
                echo "Response content: $content"
                exit 1
            fi
        else
            echo "--- Test Failed! Application returned HTTP code: $response ---"
            exit 1
        fi
    '''
}

def cleanupContainer(containerName) {
    // Clean up containers with better error handling
    try {
        sh "docker stop ${containerName} || true"
        sh "docker rm ${containerName} || true"
        echo "Successfully cleaned up container: ${containerName}"
    } catch (Exception e) {
        echo "Warning: Failed to clean up container ${containerName}: ${e.getMessage()}"
    }
}
