pipeline {
    agent any

    environment {
        NPM_CONFIG_CACHE = "${WORKSPACE}/.npm"
    }

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    rm -rf node_modules

                    npm ci
                    npm run build
                '''

                stash name: 'build-artifacts', includes: 'build/**'
                stash name: 'node-modules', includes: 'node_modules/**'
            }
        }

        stage('Test') {
            parallel {

                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        unstash 'node-modules'

                        sh '''
                            npm test -- --watchAll=false
                        '''
                    }

                    post {
                        always {
                            junit allowEmptyResults: true,
                                  testResults: 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E Tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps {
                        unstash 'build-artifacts'
                        unstash 'node-modules'

                        sh '''
                            npm install serve

                            npx serve -s build -l 3000 &
                            SERVER_PID=$!

                            sleep 10

                            npx playwright test --reporter=html

                            kill $SERVER_PID
                        '''
                    }

                    post {
                        always {
                            publishHTML([
                                allowMissing: true,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright HTML Report'
                            ])
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                unstash 'build-artifacts'

                sh '''
                    npm install netlify-cli

                    netlify --version

                    # Example deployment command
                    # netlify deploy --prod --dir=build
                '''
            }
        }
    }

    post {
        always {
            cleanWs()
        }

        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }
    }
}