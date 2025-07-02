pipeline {
    agent any

    environment {
        JIRA_BASE_URL = 'https://your-domain.atlassian.net'
        JIRA_PROJECT_KEY = 'PROJ'
        JIRA_ISSUE_TYPE = 'Bug'
        JIRA_CREDS_ID = 'jira-api-creds'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Running Build...'
                // Simulate failure
                error("Build failed for testing Jira integration")
            }
        }
    }

    post {
        failure {
            script {
                def jobName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def buildUrl = env.BUILD_URL
                def errorMsg = "Job '${jobName}' failed at build #${buildNumber}. Check details: ${buildUrl}"

                withCredentials([usernamePassword(credentialsId: env.JIRA_CREDS_ID,
                                                  usernameVariable: 'JIRA_USER',
                                                  passwordVariable: 'JIRA_TOKEN')]) {
                    def payload = """{
                        "fields": {
                            "project": {
                                "key": "${env.JIRA_PROJECT_KEY}"
                            },
                            "summary": "Jenkins Job Failed: ${jobName} #${buildNumber}",
                            "description": "${errorMsg}",
                            "issuetype": {
                                "name": "${env.JIRA_ISSUE_TYPE}"
                            }
                        }
                    }"""

                    sh """
                        curl -X POST -H "Content-Type: application/json" \
                             -u "${JIRA_USER}:${JIRA_TOKEN}" \
                             --data '${payload}' \
                             ${env.JIRA_BASE_URL}/rest/api/3/issue
                    """
                }
            }
        }
    }
}
