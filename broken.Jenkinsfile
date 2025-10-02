pipeline {
    agent any

    environment {
        DEPLOY_TARGET = 'staging-server-01'
        APP_NAME = 'FakeWebApp'
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                echo "Starting pipeline for ${APP_NAME}."
                checkout scm
            }
        }

        stage('Log Commit Information') {
            steps {
                script {
                    def commitSha = env.GIT_COMMIT ?: 'N/A'
                    def branchName = env.GIT_BRANCH ?: 'N/A'
                    def author = env.GIT_AUTHOR_NAME ?: 'Unknown Author'
                    def message = sh(returnStdout: true, script: 'git log -1 --pretty=%B').trim()

                    echo "--- Commit Details for Deployment ---"
                    echo "Application: ${APP_NAME}"
                    echo "Branch: ${branchName}"
                    echo "Commit SHA: ${commitSha.take(8)} (Full: ${commitSha})"
                    echo "Author: ${author}"
                    echo "Commit Message (First Line): ${message.tokenize('\n')[0].trim()}"
                    echo "------------------------------------"
                }
            }
        }

        stage('Build & Unit Test') {
            steps {
                echo "Running unit tests and compiling code..."
                sleep 5
                echo "Build artifact successfully created."
            }
        }

        stage('Deploy to Staging') {
            steps {
                echo "Initiating deployment to ${DEPLOY_TARGET}..."
                // Simulation of a critical deploy error (e.g., connection refusal or bad permissions)
                sh 'echo "Deployment Error: Connection refused by target server ${DEPLOY_TARGET}. Aborting deployment." && exit 1'
                echo "Deployment complete! Version: ${env.GIT_COMMIT.take(8)} is live."
            }
        }
    }

    post {
        always {
            cleanWs()
            echo "Workspace cleaned up."
        }
    }
}
