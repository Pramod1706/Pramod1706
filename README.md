pipeline {
    agent any
    stages {
        stage('Extract Version') {
            steps {
                script {
                    // Define the input string
                    def inputString = "wren api deployment of gra-wren-data-api with version 1.6.34_1.0.6"
                    
                    // Define a regular expression to extract the version number
                    def versionRegex = /(\d+\.\d+\.\d+_\d+\.\d+\.\d+)/

                    // Create matcher and extract version if it exists
                    def matcher = (inputString =~ versionRegex)
                    def version = matcher.find() ? matcher.group(0) : null // Store matched version if found

                    // Print the version
                    if (version) {
                        println "Extracted Version: ${version}"
                    } else {
                        println "No version found in the input string"
                    }
                }
            }
        }
    }
}
