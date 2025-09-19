pipeline {
    agent any
    
    stages {
        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }
        
        stage('Setup Python Environment') {
            steps {
                sh '''
                    echo "Setting up Python environment..."
                    python3 -m venv venv
                    source venv/bin/activate
                    pip install -r requirements.txt
                    pip install flask
                '''
            }
        }
        
        stage('Run Application Tests') {
            steps {
                sh '''
                    echo "Running application..."
                    source venv/bin/activate
                    
                    # Start app in background
                    nohup python3 app.py > app.log 2>&1 &
                    echo $! > app.pid
                    
                    # Wait for app to start
                    sleep 5
                    
                    # Test the application
                    if curl -s http://localhost:5000 | grep -q "Hello, DevOps World!"; then
                        echo "✅ Application test PASSED!"
                    else
                        echo "❌ Application test FAILED!"
                        cat app.log
                        exit 1
                    fi
                '''
            }
            post {
                always {
                    sh '''
                        # Stop the application
                        if [ -f app.pid ]; then
                            kill $(cat app.pid) 2>/dev/null || true
                            rm -f app.pid
                        fi
                        rm -f app.log
                    '''
                }
            }
        }
        
        stage('Build Report') {
            steps {
                sh '''
                    echo "=== BUILD SUCCESS ==="
                    echo "Application: Python Flask Microservice"
                    echo "Python Version: $(python3 --version)"
                    echo "Build Time: $(date)"
                    echo "Status: READY FOR DEPLOYMENT"
                '''
            }
        }
    }
    
    post {
        always {
            echo "Pipeline completed: ${currentBuild.result}"
        }
        success {
            echo "🎉 Pipeline executed successfully!"
        }
    }
}
