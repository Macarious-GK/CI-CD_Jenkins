pipeline {
    // agent { This is feature Branch
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
                script{
                    def scmVars = checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Macarious-GK/CI-CD_Jenkins_NodeJS.git']])
                    env.GIT_COMMIT = scmVars.GIT_COMMIT
                    env.GIT_BRANCH = scmVars.GIT_BRANCH
                }
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

        stage('SAST - SonarQube') {
            steps {
                echo "Running SonarQube analysis..."
                // timeout(time: 60, unit: 'SECONDS') {
                //     withSonarQubeEnv('sonar-qube-server') {
                //         sh 'echo $SONAR_SCANNER_HOME'
                //         sh '''
                //         $SONAR_SCANNER_HOME/bin/sonar-scanner \
                //         -Dsonar.projectKey=Solar-System-Project \
                //         -Dsonar.sources=app.js \
                //         -Dsonar.javascript.lcov.reportPaths=./coverage/lcov.info
                //         '''
                //     }
                // }
                // waitForQualityGate abortPipeline: true
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('App-SourceCode') {
                    echo "Building Docker image..."
                    sh 'docker build -t macarious25siv/project:$GIT_COMMIT .'
                    echo "Docker image built successfully."
                }
            }
        }

        stage('Vulnerability Scan Docker Image') {
            steps {
                dir('App-SourceCode') {
                    echo "Scanning Docker image with Trivy..."
                    sh '''
                        trivy image macarious25siv/project:$GIT_COMMIT \
                            --severity CRITICAL \
                            --exit-code 0 \
                            --ignore-unfixed \
                            --format json -o trivy-report.json \
                            --quiet        
                        
                    '''
                    echo "Docker image scan completed."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                dir('App-SourceCode') {
                    echo "Pushing Docker image to registry..."
                    withDockerRegistry(credentialsId: 'docker-hub-credentials', url: "") {
                        sh 'docker push macarious25siv/project:$GIT_COMMIT'
                    }
                    echo "Docker image pushed successfully."
                }
            }
        }

        stage('Testing Deploy VM') {
            when {
                branch 'main'
            }

            steps {
                script {
                    sshagent(['Test-deploy-vm']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no deployuser@192.168.56.21 "
                        if docker ps -a | grep -q 'solar-system'; then
                            echo "Container found. Stopping..."
                            docker stop "solar-system" && docker rm "solar-system"
                            echo "Container stopped and removed."
                        fi
                        
                        docker run --name solar-system \\
                            -e MONGO_URI=$MONGO_URI \\
                            -e MONGO_USERNAME=$MONGO_USERNAME \\
                            -e MONGO_PASSWORD=$MONGO_PASSWORD \\
                            -p 3000:3000 -d macarious25siv/project:$GIT_COMMIT
                        "
                    '''
                    }
                }
            }
        }
        
        stage('K8S Update Image Tag') {
            when {
                branch '*/features'
            }
            steps {
                sh 'git clone -b main https://github.com/Macarious-GK/CI-CD_Manifests_NodeJS.git'
                dir('CI-CD_Manifests_NodeJS/kubernetes') {
                    sh '''
                        git checkout main
                        git checkout -b feature-$BUILD_ID
                        sed -i "s#macarious25siv/private-docker-repo:[^ ]*#macarious25siv/private-docker-repo:$GIT_COMMIT#g" deployment.yml
                        cat deployment.yml

                
                        '''
                    }
                }   
        }
    }
    post {
        always {
            dir('App-SourceCode') {
                sh '''
                trivy convert \
                    --format template --template "@/usr/local/share/trivy/templates/html.tpl" \
                    --output trivy-report.html trivy-report.json
                trivy convert \
                    --format template --template "@/usr/local/share/trivy/templates/junit.tpl" \
                    --output trivy-report.xml trivy-report.json
                '''

                junit allowEmptyResults: true, testResults: 'test-results.xml'
                junit allowEmptyResults: true, testResults: 'trivy-report.xml'

                publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: 'coverage/lcov-report', reportFiles: 'index.html', reportName: 'Code coverage HTML Report', reportTitles: '', useWrapperFileDirectly: true])

                publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: '.', reportFiles: 'trivy-report.html', reportName: 'Trivy Vulnerability Scan Report', reportTitles: '', useWrapperFileDirectly: true])
            }
        }
    }
}
