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
	def message
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
  		script {
			if (params.FAIL_STEP) {
				error('failed on purpose.')
			}
		}
            }
        }
    }

    post {
    	success {
		sh "curl http://localhost:4000/api/rest/plugins/webhook/connections/2/deployments -X 'POST' -H 'Authorization: Bearer ' -d '{
		  \"id\": \"my-deployment-123\",
		  \"startedDate\": \"2023-01-01T12:00:00+00:00\",
		  \"finishedDate\": \"2023-01-01T12:00:00+00:00\",
		  \"result\": \"SUCCESS\",
		  \"deploymentCommits\": [
		    {
		      \"repoUrl\": \"${REPOSITORY}\",
		      \"refName\": \"${env.GIT_BRANCH}\",
		      \"startedDate\": \"2023-01-01T12:00:00+00:00\",
		      \"finishedDate\": \"2023-01-01T12:00:00+00:00\",
		      \"commitSha\": \"${env.GIT_COMMIT}\",
		      \"commitMsg\": \"${message}\"
		    }
		  ]
		}'"
	}
    	failure {
		sh "curl http://localhost:4000/api/rest/plugins/webhook/connections/2/deployments -X 'POST' -H 'Authorization: Bearer ' -d '{
		  \"id\": \"my-deployment-123\",
		  \"startedDate\": \"2023-01-01T12:00:00+00:00\",
		  \"finishedDate\": \"2023-01-01T12:00:00+00:00\",
		  \"result\": \"FAILURE\",
		  \"deploymentCommits\": [
		    {
		      \"repoUrl\": \"${REPOSITORY}\",
		      \"refName\": \"${env.GIT_BRANCH}\",
		      \"startedDate\": \"2023-01-01T12:00:00+00:00\",
		      \"finishedDate\": \"2023-01-01T12:00:00+00:00\",
		      \"commitSha\": \"${env.GIT_COMMIT}\",
		      \"commitMsg\": \"${message}\"
		    }
		  ]
		}'"
	}
        always {
            cleanWs()
            echo "Workspace cleaned up."
        }
    }
}
