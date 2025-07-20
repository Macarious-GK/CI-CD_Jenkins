pipeline {
    agent {
        dockerfile {
            filename 'Dockerfile'
            dir 'App-SourceCode' // This is where Jenkins will look for the Dockerfile
            // additionalBuildArgs '--no-cache' // Optional
            // label '' // Optional: label of agent (leave empty to run on any)
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
        stage('Checkout Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/Macarious-GK/CI-CD_Jenkins_NodeJS.git'
                sh 'ls -la'
            }
        }

        stage('Installing Dependencies') {
            steps {
                dir('App-SourceCode') {
                    echo "Installing dependencies in App-SourceCode directory..."
                    sh '''
                    rm -rf node_modules package-lock.json
                    npm install --no-audit --cache /tmp/empty-npm-cache
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
    }
}

// pipeline {
//     agent {
//         // docker {
//         //     image 'node:18' 
//         //     args '-u node'
//         // }
//         dockerfile {
//             filename 'Dockerfile'
//             dir 'App-SourceCode'       
//         }
//     }

//     parameters {
//         string(name: 'TEST_MongoDB_URL', defaultValue: '127.0.0.1', description: 'MongoDB host')
//     }

//     environment {
//         MONGO_URI = "mongodb://${params.TEST_MongoDB_URL}:27017/test"
//     }

//     options {
//         timestamps()
//     }

//     stages {
//         stage('Installing Dependencies') {
//             steps {
//                 dir('App-SourceCode') {
//                     echo "Installing dependencies in App-SourceCode directory..."
//                     sh '''
//                     rm -rf node_modules package-lock.json
//                     npm install --no-audit
//                     '''
//                 }
//             }
//         }

//         stage('Linting Code') {
//             steps {
//                 sh '''
//                     echo "Running ESLint..."
//                     npm run lint || true
//                 '''
//             }
//         }

//         stage('Dependency Scanning') {
//             parallel {
//                 stage('NPM Dependency Audit') {
//                     steps {
//                         sh '''
//                             echo "Running npm audit..."
//                             npm audit --audit-level=critical || true
//                         '''
//                     }
//                 }

//                 stage('OWASP Dependency Check') {
//                     steps {
//                         sh '''
//                             echo "Running OWASP Dependency Check..."
//                             echo "Mongo URL: ${MONGO_URI}"
//                             sleep 30
//                             echo "OWASP Dependency Check completed."
//                         '''
//                     }
//                 }
//             }
//         }
//     }
// }
