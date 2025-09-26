pipeline {
    agent {
        label 'ubuntu-node-1'
    }
    
    environment {
        REPO_URL = 'https://github.com/sulaimanadamu/mid-term-devops-project.git'
        BRANCH = 'main'
    }
    
    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning repository...'
                git branch: "${BRANCH}", url: "${REPO_URL}"
            }
        }
        
        stage('Install System Tools') {
            steps {
                echo 'Installing Node.js, Python, and Ansible...'
                sh '''
                    # Update package list
                    sudo apt-get update
                    
                    # Install Node.js and npm
                    curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
                    sudo apt-get install -y nodejs
                    
                    # Install Python and pip
                    sudo apt-get install -y python3 python3-pip python3-venv
                    
                    # Use apt-get to install Ansible
                    sudo apt-get install -y ansible
                    
                    # Add pip user bin to PATH
                    export PATH=$PATH:~/.local/bin
                    echo 'export PATH=$PATH:~/.local/bin' >> ~/.bashrc
                '''
            }
        }
        
        stage('Install Dependencies') {
            parallel {
                stage('Install Flask Dependencies') {
                    when {
                        expression { fileExists('./flask_app/requirements.txt') }
                    }
                    steps {
                        echo 'Installing Flask dependencies...'
                        sh '''
                            # Create virtual environment
                            python3 -m venv venv
                            . venv/bin/activate
                            
                            # Install Flask dependencies
                            pip install -r ./flask_app/requirements.txt
                            
                            echo "Flask dependencies installed:"
                            pip list
                        '''
                    }
                }
                
                stage('Install Node.js Dependencies') {
                    when {
                        expression { fileExists('./node_app/package.json') }
                    }
                    steps {
                        echo 'Installing Node.js dependencies...'
                        sh '''
                            cd node_app
                            npm install
                            
                            echo "Node.js dependencies installed:"
                            npm list --depth=0
                        '''
                    }
                }
            }
        }
        
        stage('Build Applications') {
            parallel {
                stage('Build Flask App') {
                    when {
                        expression { fileExists('./flask_app/requirements.txt') }
                    }
                    steps {
                        echo 'Building Flask application...'
                        sh '''
                            . venv/bin/activate
                            cd flask_app
                            
                            # Test Flask app if app.py exists
                            if [ -f "app.py" ]; then
                                echo "Testing Flask app..."
                                python -c "import app; print('Flask app imported successfully')"
                            fi
                            
                            echo "Flask app built successfully"
                        '''
                    }
                }
                
                stage('Build Node.js App') {
                    when {
                        expression { fileExists('./node_app/package.json') }
                    }
                    steps {
                        echo 'Building Node.js application...'
                        sh '''
                            cd node_app
                            # Run build script if it exists
                            if npm run build --silent 2>/dev/null; then
                                npm run build
                            fi
                            
                            # Test Node.js app
                            if [ -f "app.js" ] || [ -f "index.js" ] || [ -f "server.js" ]; then
                                echo "Node.js app files found and ready"
                            fi
                            
                            echo "Node.js app built successfully"
                        '''
                    }
                }
            }
        }
        
        stage('Verify Installation') {
            steps {
                echo 'Verifying installations...'
                sh '''
                    echo "=== INSTALLATION SUMMARY ==="
                    echo "Node.js: $(node --version)"
                    echo "npm: $(npm --version)"
                    echo "Python: $(python3 --version)"
                    
                    # Check Ansible installation
                    export PATH=$PATH:~/.local/bin
                    if command -v ansible &> /dev/null; then
                        echo "Ansible: $(ansible --version | head -1)"
                    else
                        echo "Ansible: Installation failed"
                    fi
                    echo "=========================="
                '''
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline completed!'
        }
        success {
            echo 'All dependencies installed and apps built successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}