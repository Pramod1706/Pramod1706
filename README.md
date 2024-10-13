pipeline {
    agent any
    stages {
        stage('Get Crumb') {
            steps {
                script {
                    def response = httpRequest(
                        url: "http://your-jenkins-url/crumbIssuer/api/json",
                        authentication: 'jenkins-credentials-id'
                    )
                    def json = readJSON text: response.content
                    echo "Crumb: ${json.crumb}"
                }
            }
        }
    }
}
