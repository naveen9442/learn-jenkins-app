pipeline {
    agent any

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
                    export NPM_CONFIG_CACHE=$WORKSPACE/.npm

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
                            export NPM_CONFIG_CACHE=$WORKSPACE/.npm

                            npm test -- --watchAll=false
                        '''
                    }

                    post {
                        always {
                            junit(
                                allowEmptyResults: true,
                                testResults: 'jest-results/junit.xml'
                            )
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
                            export NPM_CONFIG_CACHE=$WORKSPACE/.npm

                            npm install serve

                            npx serve -s build -l 3000 &
                            SERVER_PID=$!

                            sleep 10

                            npx playwright test --reporter=html

                            kill $SERVER_PID || true
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
                    export NPM_CONFIG_CACHE=$WORKSPACE/.npm

                    mkdir -p $NPM_CONFIG_CACHE

                    npx netlify-cli --version

                    # Example deployment:
                    # npx netlify-cli deploy \
                    #   --dir=build \
                    #   --prod \
                    #   --auth=$NETLIFY_AUTH_TOKEN \
                    #   --site=$NETLIFY_SITE_ID
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
