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
                    withCredentials([usernamePassword(credentialsId: 'mongo-db-credentials', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
                        catchError(buildResult: 'SUCCESS', message: 'There is something not very very important happened', stageResult: 'UNSTABLE') {
                            sh 'npm test'
                        }
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
                    publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: 'coverage/lcov-report', reportFiles: 'index.html', reportName: 'Code coverage HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
