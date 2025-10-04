pipeline {
    agent any

    parameters {
        booleanParam(name: 'SHOULD_PASS', defaultValue: true, description: 'Defines if the pipeline should pass successfully or not')
    }

    environment {
        DEPLOY_TARGET = 'staging-server-01'
        APP_NAME = 'FakeWebApp'
        REPOSITORY = 'https://github.com/joseiedo/devlake-version-control-tester'
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
                def now = new Date().format("yyyy-MM-dd'T'HH:mm:ssZ", java.util.TimeZone.getTimeZone('UTC'))
                def deployEndTime = new Date().format("yyyy-MM-dd'T'HH:mm:ssZ", java.util.TimeZone.getTimeZone('UTC'))

                sh """curl http://devlake-config-ui-1:4000/api/rest/plugins/webhook/connections/2/deployments -X 'POST' -H 'Authorization: Bearer eLgjJAH5wN0zlboo26oIfh9zLh3sAQiv5iBTRx2terkvg9U8JFfuuTFDu0m8sK3FVWvBw0yHTqIJofUpWLfVRIQKD2XrBhkHAhGyKseHcCi9lDooDJkQ2k4I9VwLZyU2' -d '{
                    \"id\": \"jenkins:JenkinsBuild:2:deploy#${env.BUILD_NUMBER}\",
                    \"startedDate\": \"${now}\",
                    \"finishedDate\": \"${deployEndTime}\",
                    \"result\": \"SUCCESS\",
                    \"deploymentCommits\": [
                        {
                            \"repoUrl\": \"${REPOSITORY}\",
                            \"refName\": \"${env.GIT_BRANCH}\",
                            \"startedDate\": \"${now}\",
                            \"finishedDate\": \"${deployEndTime}\",
                            \"commitSha\": \"${env.GIT_COMMIT}\",
                            \"commitMsg\": \"deployment sucess\"
                        }
                    ]
                }'"""
            }
        }
        failure {
            script {
                def now = new Date().format("yyyy-MM-dd'T'HH:mm:ssZ", java.util.TimeZone.getTimeZone('UTC'))
                def deployEndTime = new Date().format("yyyy-MM-dd'T'HH:mm:ssZ", java.util.TimeZone.getTimeZone('UTC'))

                sh """curl http://devlake-config-ui-1:4000/api/rest/plugins/webhook/connections/2/deployments -X 'POST' -H 'Authorization: Bearer eLgjJAH5wN0zlboo26oIfh9zLh3sAQiv5iBTRx2terkvg9U8JFfuuTFDu0m8sK3FVWvBw0yHTqIJofUpWLfVRIQKD2XrBhkHAhGyKseHcCi9lDooDJkQ2k4I9VwLZyU2' -d '{
                    \"id\": \"jenkins:JenkinsBuild:2:deploy#${env.BUILD_NUMBER}\",
                    \"startedDate\": \"${now}\",
                    \"finishedDate\": \"${deployEndTime}\",
                    \"result\": \"FAILURE\",
                    \"deploymentCommits\": [
                        {
                            \"repoUrl\": \"${REPOSITORY}\",
                            \"refName\": \"${env.GIT_BRANCH}\",
                            \"startedDate\": \"${now}\",
                            \"finishedDate\": \"${deployEndTime}\",
                            \"commitSha\": \"${env.GIT_COMMIT}\",
                            \"commitMsg\": \"deployment failure\"
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
