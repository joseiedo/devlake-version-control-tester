pipeline {
    agent any

    parameters {
        booleanParam(name: 'SHOULD_PASS', defaultValue: true, description: 'Defines if the pipeline should pass successfully or not')
    }

    environment {
        DEPLOY_TARGET = 'staging-server-01'
        APP_NAME = 'FakeWebApp'
        REPOSITORY = 'https://github.com/joseiedo/devlake-version-control-tester'
        
        // Dynamic variables initialized here, values set in stages
        DEPLOY_START_TIME = ''
        COMMIT_START_TIME = ''
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
                    
                    def COMMIT_MESSAGE = sh(returnStdout: true, script: 'git log -1 --pretty=%B').trim()

                    echo "--- Commit Details for Deployment ---"
                    echo "Application: ${APP_NAME}"
                    echo "Branch: ${branchName}"
                    echo "Commit SHA: ${commitSha.take(8)} (Full: ${commitSha})"
                    echo "Author: ${author}"
                    echo "Commit Message (First Line): ${COMMIT_MESSAGE.tokenize('\n')[0].trim()}"
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
                script {
                    // --- 1. Capture the START time immediately before deployment begins ---
                    // Format: yyyy-MM-dd'T'HH:mm:ssXXX (e.g., 2023-01-01T12:00:00+00:00)
                    def now = java.time.ZonedDateTime.now(java.time.ZoneOffset.UTC).format(java.time.format.DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ssXXX"))
                    
                    // Assign to global variables for use in the post block
                    env.DEPLOY_START_TIME = now
                    env.COMMIT_START_TIME = now // Assuming commit process starts now too

                    // Using a parameter check based on your provided structure
                    if (!params.SHOULD_PASS) {
                        error('Failed on purpose as requested by parameter.')
                    } else {
                        echo "Deployment successful to ${DEPLOY_TARGET}."
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                // --- 2. Capture the END time when the pipeline finishes successfully ---
                def deployEndTime = java.time.ZonedDateTime.now(java.time.ZoneOffset.UTC).format(java.time.format.DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ssXXX"))

                sh """curl http://devlake-config-ui-1:4000/api/rest/plugins/webhook/connections/2/deployments -X 'POST' -H 'Authorization: Bearer eLgjJAH5wN0zlboo26oIfh9zLh3sAQiv5iBTRx2terkvg9U8JFfuuTFDu0m8sK3FVWvBw0yHTqIJofUpWLfVRIQKD2XrBhkHAhGyKseHcCi9lDooDJkQ2k4I9VwLZyU2' -d '{
                    \"id\": \"jenkins:JenkinsBuild:2:deploy#${env.BUILD_NUMBER}\",
                    \"startedDate\": \"${env.DEPLOY_START_TIME}\",
                    \"finishedDate\": \"${deployEndTime}\",
                    \"result\": \"SUCCESS\",
                    \"deploymentCommits\": [
                        {
                            \"repoUrl\": \"${REPOSITORY}\",
                            \"refName\": \"${env.GIT_BRANCH}\",
                            // Use the dynamic deployment times for the commit as well
                            \"startedDate\": \"${env.COMMIT_START_TIME}\",
                            \"finishedDate\": \"${deployEndTime}\",
                            \"commitSha\": \"${env.GIT_COMMIT}\",
                            \"commitMsg\": \"deployment made :)\"
                        }
                    ]
                }'"""
            }
        }
        failure {
            script {
                // --- 2. Capture the END time when the pipeline finishes in failure ---
                def deployEndTime = java.time.ZonedDateTime.now(java.time.ZoneOffset.UTC).format(java.time.format.DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ssXXX"))

                sh """curl http://devlake-config-ui-1:4000/api/rest/plugins/webhook/connections/2/deployments -X 'POST' -H 'Authorization: Bearer eLgjJAH5wN0zlboo26oIfh9zLh3sAQiv5iBTRx2terkvg9U8JFfuuTFDu0m8sK3FVWvBw0yHTqIJofUpWLfVRIQKD2XrBhkHAhGyKseHcCi9lDooDJkQ2k4I9VwLZyU2' -d '{
                    \"id\": \"jenkins:JenkinsBuild:2:deploy#${env.BUILD_NUMBER}\",
                    \"startedDate\": \"${env.DEPLOY_START_TIME}\",
                    \"finishedDate\": \"${deployEndTime}\",
                    \"result\": \"FAILURE\",
                    \"deploymentCommits\": [
                        {
                            \"repoUrl\": \"${REPOSITORY}\",
                            \"refName\": \"${env.GIT_BRANCH}\",
                            // Use the dynamic deployment times for the commit as well
                            \"startedDate\": \"${env.COMMIT_START_TIME}\",
                            \"finishedDate\": \"${deployEndTime}\",
                            \"commitSha\": \"${env.GIT_COMMIT}\",
                            \"commitMsg\": \"deployment made :)\"
                        }
                    ]
                }'"""
            }
        }
        always {
            cleanWs()
            echo "Workspace cleaned up."
        }
    }
}
