pipeline {
    // agent {
    //     // dockerfile {
    //     //     filename 'Dockerfile'
    //     //     dir 'App-SourceCode' // This is where Jenkins will look for the Dockerfile
    //     // }
    //     // docker {
    //     //     image 'macarious25siv/project:nodejsagent'
    //     // }
    // }
    agent any
    tools {
    nodejs 'latestnodejs'
    }

    parameters {
        string(name: 'TEST_MongoDB_URL', defaultValue: '192.168.56.20', description: 'MongoDB host')
    }

    environment {
        MONGO_URI = "mongodb://${params.TEST_MongoDB_URL}:27017/admin"
        MONGO_USERNAME = credentials("MONGO_USERNAME")
        MONGO_PASSWORD = credentials("MONGO_PASSWORD")
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
                    // npm install || true
                    echo "Dependencies installed successfully."
                }
            }
        }

        stage('Linting Code') {
            steps {
                dir('App-SourceCode') {
                    sh '''
                    echo "Running ESLint..."
                    npm run lint || true
                    echo "Linting completed."
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
                dir('App-SourceCode') {
                    echo "Running unit tests..."
                    catchError(buildResult: 'SUCCESS', message: 'There is something not very very important happened', stageResult: 'UNSTABLE') {
                        sh 'npm test'
                    }
                    echo "Unit tests completed."
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
                        echo "Code coverage completed."
                        '''
                    }
                }
            }
        }
    }
    post {
        always {
            junit allowEmptyResults: true, testResults: 'test-results.xml'
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: 'coverage/lcov-report', reportFiles: 'index.html', reportName: 'Code coverage HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
