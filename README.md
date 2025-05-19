# Python-code

pipeline {
    agent any

    environment {
        JENKINS_USER = 'your-username'
        JENKINS_PASS = 'your-password'
        JOB_URL = 'http://jenkins.local/job/your-job-name/lastBuild/api/json'
    }

    stages {
        stage('Generate Job Details Report') {
            steps {
                script {
                    def authString = "${env.JENKINS_USER}:${env.JENKINS_PASS}".bytes.encodeBase64().toString()
                    def url = new URL(env.JOB_URL)
                    def connection = url.openConnection()
                    connection.setRequestProperty("Authorization", "Basic ${authString}")
                    def response = connection.inputStream.text

                    def json = new groovy.json.JsonSlurper().parseText(response)

                    def buildNumber = json.number
                    def buildStatus = json.result
                    def buildTime = new Date(json.timestamp).toString()
                    def paramList = []

                    json.actions.each { action ->
                        if (action?.parameters) {
                            action.parameters.each { param ->
                                def value = param.value
                                paramList << [
                                    name: param.name,
                                    value: value,
                                    type: value?.getClass()?.simpleName ?: 'null',
                                    length: value?.toString()?.length() ?: 0,
                                    isEmpty: (value == null || value.toString().trim().isEmpty()),
                                    display: value?.toString() ?: ''
                                ]
                            }
                        }
                    }

                    def paramTableRows = paramList.collect {
                        """
                        <tr>
                            <td>${it.name}</td>
                            <td>${it.value}</td>
                            <td>${it.type}</td>
                            <td>${it.length}</td>
                            <td>${it.isEmpty}</td>
                            <td>${it.display}</td>
                        </tr>
                        """
                    }.join("\n")

                    def reportHtml = """
                        <html>
                        <head>
                            <title>Jenkins Job Report</title>
                            <style>
                                table { border-collapse: collapse; width: 100%; }
                                th, td { border: 1px solid #ddd; padding: 8px; }
                                th { background-color: #f2f2f2; text-align: left; }
                            </style>
                        </head>
                        <body>
                            <h2>Job: ${json.fullDisplayName}</h2>
                            <p><strong>Build Number:</strong> ${buildNumber}</p>
                            <p><strong>Status:</strong> ${buildStatus}</p>
                            <p><strong>Timestamp:</strong> ${buildTime}</p>

                            <h3>Parameter Details</h3>
                            <table>
                                <thead>
                                    <tr>
                                        <th>Name</th>
                                        <th>Value</th>
                                        <th>Type</th>
                                        <th>Length</th>
                                        <th>Is Empty</th>
                                        <th>ToString()</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    ${paramTableRows}
                                </tbody>
                            </table>
                        </body>
                        </html>
                    """

                    writeFile file: 'job-report.html', text: reportHtml
                    archiveArtifacts artifacts: 'job-report.html', onlyIfSuccessful: true

                    emailext(
                        subject: "Jenkins Job Report - ${json.fullDisplayName}",
                        to: 'you@example.com',
                        mimeType: 'text/html',
                        body: reportHtml
                    )
                }
            }
        }
    }
}
