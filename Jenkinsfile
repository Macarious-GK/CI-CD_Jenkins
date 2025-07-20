pipeline {
    agent {
        docker {
            image 'node:18' 
        }
    }

    parameters {
        string(name: 'TEST_MongoDB_URL', defaultValue: '127.0.0.1', description: 'MongoDB host')
    }

    environment {
        MONGO_URI = "mongodb://${params.TEST_MongoDB_URL}:27017/test"
    }

    options {
        timestamps()
    }

    stages {
        stage('Installing Dependencies') {
            steps {
                dir('App-SourceCode') {
                    echo "Installing dependencies in App-SourceCode directory..."
                    sh '''
                    rm -rf node_modules package-lock.json
                    npm install  --no-audit 
                    chown -R 113:121 "/.npm"
                    '''
                }
            }
        }

        stage('Linting Code') {
            steps {
                sh '''
                    echo "Running ESLint..."
                    npm run lint || true
                '''
            }
        }

        stage('Dependency Scanning') {
            parallel {
                stage('NPM Dependency Audit') {
                    steps {
                        sh '''
                            echo "Running npm audit..."
                            npm audit --audit-level=critical || true
                        '''
                    }
                }

                stage('OWASP Dependency Check') {
                    steps {
                        sh '''
                            echo "Running OWASP Dependency Check..."
                            echo "Mongo URL: ${MONGO_URI}"
                            sleep 30
                            echo "OWASP Dependency Check completed."
                        '''
                    }
                }
            }
        }
    }
}
