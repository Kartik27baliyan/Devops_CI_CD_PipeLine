pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                checkout scm // This automatically checks out the code from the repository URL defined in the job
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image, tagging it with the build number
                    dockerImage = docker.build("kartik-python-app:${env.BUILD_ID}")
                }
            }
        }

        stage('Test Docker Image') {
            steps {
                script {
                    // Run the container in the background
                    dockerContainer = dockerImage.run("-p 5000:5000 -d --name test-container-${env.BUILD_ID}")
                    // Wait a few seconds for the app to start
                    sleep(5)
                    // Test if the application is running by curling localhost
                    sh '''
                        if curl -s http://localhost:5000 | grep "Hello, DevOps World!"; then
                            echo "--- Test Passed! Application is running correctly. ---"
                        else
                            echo "--- Test Failed! Application did not return the expected text. ---"
                            exit 1 // This fails the build
                        fi
                    '''
                }
            }
            post {
                always {
                    // Clean up the container after the test, whether it passes or fails
                    script {
                        try {
                            sh "docker stop test-container-${env.BUILD_ID}"
                            sh "docker rm test-container-${env.BUILD_ID}"
                        } catch (Exception e) {
                            echo "Failed to clean up test container: ${e}"
                        }
                    }
                }
            }
        }
    }
}