
pipeline {
    // Defines where the pipeline runs (e.g., 'any' agent or a specific label 'docker')
    agent any

    // Environment variables that apply to the entire pipeline
    environment {
        // Example variables (you might set credentials or configuration paths here)
        DEPLOY_TARGET = 'staging-server-01'
        APP_NAME = 'FakeWebApp'
    }

    stages {
        // --- Stage 1: Checkout and Setup ---
        stage('Checkout Source Code') {
            steps {
                echo "Starting pipeline for ${APP_NAME}."
                // The 'checkout scm' step uses the configuration from the job definition
                // to clone the repository (which automatically populates GIT environment variables).
                checkout scm
            }
        }

        // --- Stage 2: Get Commit Details (The Core Request) ---
        stage('Log Commit Information') {
            // This stage demonstrates how to access commit details after checkout.
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

        // --- Stage 3: Build and Test (Simulated) ---
        stage('Build & Unit Test') {
            steps {
                echo "Running unit tests and compiling code..."
                // Replace with actual build command (e.g., sh 'npm install && npm run build')
                sleep 5 // Simulate build time
                echo "Build artifact successfully created."
            }
        }

        // --- Stage 4: Deploy to Staging (Simulated) ---
        stage('Deploy to Staging') {
            when {
                // Only runs if the current branch is 'main' or 'master'
                expression { return env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'master' }
            }
            steps {
                echo "Initiating deployment to ${DEPLOY_TARGET}..."
                // Replace with actual deployment logic (e.g., sh 'rsync -avz ./dist user@${DEPLOY_TARGET}:/var/www/${APP_NAME}')
                sleep 10 // Simulate deployment time
                echo "Deployment complete! Version: ${env.GIT_COMMIT.take(8)} is live."
            }
        }

        // --- Stage 5: Notifications ---
        stage('Post-Deployment Notification') {
            steps {
                echo "Sending success notification to relevant channels."
                // Implement notification logic here (e.g., Slack, Email)
            }
        }
    }

    // Optional post-build actions, regardless of stage success/failure
    post {
        always {
            // Clean up workspace to save space
            cleanWs()
            echo "Workspace cleaned up."
        }
    }
}