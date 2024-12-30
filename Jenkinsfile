pipeline {
    agent any

    parameters {
        string(name: 'ENV', defaultValue: 'development', description: 'Environment to deploy to')
    }

    environment {
        NODE_ENV = "${params.ENV}"
    }

    stages {
        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies...'
                sh 'npm install'
                // cache(path: 'node_modules', key: "npm-cache-${env.BRANCH_NAME}") {
                //     sh 'npm install'
                // }
            }
        }

        stage('Code Quality Checks') {
            parallel {
                stage('Format') {
                    steps {
                        echo 'Running Prettier to format code...'
                        sh 'npm run format'
                    }
                }
                stage('Lint') {
                    steps {
                        echo 'Linting the code...'
                        sh 'npm run lint'
                    }
                }
                stage('Type Check') {
                    steps {
                        echo 'Checking TypeScript types...'
                        sh 'npm run type-check'
                    }
                }
            }
        }

        stage('Security and Analysis') {
            parallel {
                stage('Security Scan') {
                    steps {
                        echo 'Running security scan...'
                        sh 'npm audit'
                    }
                }
                stage('Static Code Analysis') {
                    steps {
                        echo 'Running static code analysis...'
                        sh 'sonar-scanner'
                    }
                }
            }
        }

        stage('Advanced Security Scan') {
            steps {
                sh 'snyk test'
                sh 'zap-cli quick-scan http://localhost:3000'
            }
        }

        stage('Build') {
            steps {
                echo 'Building the project...'
                sh 'npm run build'
            }
        }

        stage('Testing') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        echo 'Running unit tests...'
                        sh 'npm test'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        echo 'Running integration tests...'
                        sh 'npm run integration-test'
                    }
                }
            }
        }

        stage('Serve') {
            steps {
                echo 'Serving the application in development mode...'
                sh 'npm run dev &' // Use npm run start for production
                sleep time: 10, unit: 'SECONDS'
            }
        }

        stage('Lighthouse CI') {
            steps {
                echo 'Running Lighthouse CI...'
                sh './node_modules/.bin/lhci autorun'
            }
        }

        stage('Performance Testing') {
            steps {
                echo 'Running performance tests...'
                sh 'npm run performance-test'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t my-nextjs-app .'
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'out/**', fingerprint: true
            }
        }

        stage('Deploy') {
            when {
                expression { params.ENV == 'production' }
            }
            steps {
                echo 'Deploying the application...'
                sh 'npm run deploy'
            }
        }

        stage('Notify') {
            steps {
                script {
                    def buildStatus = currentBuild.currentResult
                    def buildUrl = env.BUILD_URL
                    def branchName = env.BRANCH_NAME
                    def message = "Build ${env.BUILD_NUMBER} for branch ${branchName} finished with status: ${buildStatus}. See details at ${buildUrl}"
                    slackSend channel: '#build-notifications', message: message
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
            archiveArtifacts artifacts: 'logs/**', allowEmptyArchive: true
            mail to: 'dev-team@example.com',
             subject: "Build ${env.BUILD_NUMBER} Failed",
             body: "Check the logs at ${env.BUILD_URL}"
        }
        always {
            echo 'Cleaning up...'
            cleanWs()
            publishHTML(target: [
                reportDir: 'coverage',
                reportFiles: 'index.html',
                reportName: 'Coverage Report'
            ])
        }
    }
}
