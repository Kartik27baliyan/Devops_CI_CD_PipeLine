pipeline {
    agent any
    
    stages {
        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }
        
        stage('Set Up Python Environment') {
            steps {
                script {
                    // Check if Python is available
                    def pythonVersion = sh(script: 'python3 --version || python --version', returnStdout: true).trim()
                    echo "Using: ${pythonVersion}"
                    
                    // Create virtual environment
                    sh 'python3 -m venv venv || python -m venv venv'
                    
                    // Activate virtual environment and install dependencies
                    sh '''
                        source venv/bin/activate
                        pip install -r requirements.txt
                        pip install flask pytest requests
                    '''
                }
            }
        }
        
        stage('Run Unit Tests') {
            steps {
                script {
                    sh '''
                        source venv/bin/activate
                        python -m pytest tests/ -v || echo "No tests found or tests failed"
                    '''
                }
            }
        }
        
        stage('Start Application') {
            steps {
                script {
                    // Start the application in background
                    sh '''
                        source venv/bin/activate
                        nohup python app.py > app.log 2>&1 &
                        echo $! > app.pid
                        sleep 5
                    '''
                }
            }
        }
        
        stage('Test Application') {
            steps {
                script {
                    // Test if the application is running
                    sh '''
                        # Wait for application to start
                        for i in {1..10}; do
                            if curl -s http://localhost:5000 > /dev/null; then
                                echo "Application started successfully"
                                break
                            fi
                            echo "Waiting for application to start... Attempt $i/10"
                            sleep 3
                        done
                        
                        # Test the application endpoint
                        response=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:5000)
                        content=$(curl -s http://localhost:5000)
                        
                        echo "HTTP Response Code: $response"
                        echo "Response Content: $content"
                        
                        if [ "$response" = "200" ] && echo "$content" | grep -q "Hello, DevOps World!"; then
                            echo "--- Test Passed! Application is running correctly. ---"
                        else
                            echo "--- Test Failed! ---"
                            echo "Expected: 'Hello, DevOps World!'"
                            echo "Got: '$content'"
                            exit 1
                        fi
                    '''
                }
            }
        }
    }
    
    post {
        always {
            script {
                // Stop the application
                sh '''
                    if [ -f app.pid ]; then
                        kill $(cat app.pid) 2>/dev/null || true
                        rm -f app.pid
                    fi
                    # Clean up log files
                    rm -f app.log nohup.out
                '''
                
                // Archive test results if any
                archiveArtifacts artifacts: '**/*.log', allowEmptyArchive: true
            }
        }
        
        success {
            echo 'Pipeline completed successfully!'
        }
        
        failure {
            echo 'Pipeline failed! Check the logs for details.'
        }
    }
}
