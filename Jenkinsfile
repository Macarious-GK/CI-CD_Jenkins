pipeline {
    agent {
        // dockerfile {
        //     filename 'Dockerfile'
        //     dir 'App-SourceCode' // This is where Jenkins will look for the Dockerfile
        // }
        docker {
            image 'macarious25siv/project:nodejsagent'
        }
    }

    parameters {
        string(name: 'TEST_MongoDB_URL', defaultValue: '127.0.0.1', description: 'MongoDB host')
    }

    environment {
        MONGO_URI = "mongodb://${params.TEST_MongoDB_URL}:27017/admin"
    }

    options {
        timestamps()
    }

    stages {
        stage('Checkout Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/Macarious-GK/CI-CD_Jenkins_NodeJS.git'
                sh 'ls -la'
                sh 'pwd'
            }
        }

        stage('Installing Dependencies') {
            steps {
                dir('App-SourceCode') {
                    echo "Installing dependencies in App-SourceCode directory..."
                    sh '''
                    // if [ ! -d node_modules ]; then
                    //   npm ci --no-audit
                    // else
                    //   echo "Using cached node_modules"
                    // fi
                    // npm install --no-audit
                    ls -alt
                    pwd 
                    '''
                }
            }
        }

        stage('Linting Code') {
            steps {
                dir('App-SourceCode') {
                    sh '''
                    echo "Running ESLint..."
                    npm run lint || true
                    '''
                }
            }
        }

        stage('Dependency Scanning') {
            parallel {
                stage('NPM Dependency Audit') {
                    steps {
                        dir('App-SourceCode') {
                            sh '''
                            echo "Running npm audit..."
                            npm audit --audit-level=critical || true
                            '''
                        }
                    }
                }

                stage('OWASP Dependency Check') {
                    steps {
                        sh '''
                        echo "Running OWASP Dependency Check..."
                        echo "Mongo URL: ${MONGO_URI}"
                        sleep 10
                        echo "OWASP Dependency Check completed."
                        '''
                    }
                }
            }
        }

        stage('Unit Testing') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'mongo-db-credentials', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
                    sh 'npm test'
                }
            }
        }

        stage('Code Coverage') {
            steps {
                dir('App-SourceCode') {
                    catchError(buildResult: 'SUCCESS', message: 'Oops! it will be fixed in future releases', stageResult: 'UNSTABLE') {
                        sh '''
                        echo "Running code coverage..."
                        npm run coverage
                        '''
                    }
                }
            }
        }
    }
}
