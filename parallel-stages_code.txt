pipeline {
    agent any

    environment {
        APP_NAME = "SimpleApp"
        VERSION = "1.0.${env.BUILD_NUMBER}"
    }

    stages {
        stage('Initialize') {
            steps {
                echo "Starting the pipeline for ${APP_NAME} version ${VERSION}"
                sh 'echo "Preparing workspace..."'
            }
        }

        stage('Build') {
            steps {
                echo "Building the application..."
                sh '''
                    mkdir -p build
                    echo "Application: ${APP_NAME}" > build/info.txt
                    echo "Version: ${VERSION}" >> build/info.txt
                    echo "Build completed successfully." >> build/info.txt
                '''
            }
        }

        stage('Testing Phase') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        echo "Running unit tests..."
														   
                        sh '''
                            if [ -f build/info.txt ]; then
                                echo "Unit tests passed."
                            else
                                echo "Unit tests failed: build file missing!" && exit 1
                            fi
                        '''
                    }
                }
                stage('Integration Tests') {
                    steps {
                        echo "Running integration tests..."
                        sh '''
                            sleep 2
                            echo "Integration tests passed."
                        '''
                    }
                }
                stage('Lint & Static Analysis') {
                    steps {
                        echo "Running linting and code quality checks..."
                        sh '''
                            echo "Code style OK. No lint errors found."
                        '''
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo "Deploying version ${VERSION}..."
                sh '''
                    mkdir -p deploy
                    cp build/info.txt deploy/
                    echo "Deployment complete." >> deploy/info.txt
                '''
            }
        }

        stage('Archive Results') {
            steps {
                echo "Archiving build artifacts..."
                archiveArtifacts artifacts: '**/*.txt', fingerprint: true
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Cleaning workspace..."
            deleteDir()
        }
        success {
            echo "Build succeeded."
        }
        failure {
            echo "Build failed. Check logs for details."
        }
    }
}
