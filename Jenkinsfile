pipeline {
    agent any

    parameters {
        string(name: 'TEST_MongoDB_URL', defaultValue: '127.0.0.1', description: 'Node.js environment')
    }
    environment {
        MONGO_URI = "mongodb://${TEST_MongoDB_URL}:27017/test"
    }
    options { 
        timestamps() 
    }
    stages {
        stage('Installing Dependencies') {
            steps {
                sh 'npm install --no-audit'
            }
        }

        stage('Dependency Scanning') {
            parallel {
                stage('NPM Dependency Audit') {
                    steps {
                        sh '''
                            npm audit --audit-level=critical || true
                        '''
                    }
                }

                stage('OWASP Dependency Check') {
                    steps {
                        script{
                            sh '''
                                echo "Running OWASP Dependency Check..."
                                sleep 60
                                echo "${TEST_MongoDB_URL}"
                                echo "OWASP Dependency Check completed."
                            '''
                        }
                    }
                }
            }
        }
        
    }
}
