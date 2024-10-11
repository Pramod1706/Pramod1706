Below is an example of a Jenkinsfile that downloads a .tar.gz file from a Nexus repository, extracts it, and then pushes the content to another Nexus repository. This Jenkinsfile uses shell commands and assumes you are using Maven to interact with Nexus. Modify it according to your environment and credentials.

pipeline {
    agent any

    environment {
        NEXUS_DOWNLOAD_REPO_URL = 'http://<nexus-url>/repository/<download-repo>/'
        NEXUS_UPLOAD_REPO_URL = 'http://<nexus-url>/repository/<upload-repo>/'
        NEXUS_USERNAME = credentials('nexus-username') // Jenkins credentials ID for Nexus username
        NEXUS_PASSWORD = credentials('nexus-password') // Jenkins credentials ID for Nexus password
        FILE_NAME = '<file-name>.tar.gz'
        GROUP_ID = '<group-id>' // Group ID for upload
        ARTIFACT_ID = '<artifact-id>' // Artifact ID for upload
        VERSION = '<version>' // Version of the artifact
        PACKAGING = 'tar.gz' // File type
    }

    stages {
        stage('Download .tar.gz from Nexus') {
            steps {
                script {
                    sh """
                    # Download the tar.gz file from Nexus
                    wget --user=${NEXUS_USERNAME} --password=${NEXUS_PASSWORD} ${NEXUS_DOWNLOAD_REPO_URL}${FILE_NAME}
                    """
                }
            }
        }

        stage('Extract the .tar.gz file') {
            steps {
                script {
                    sh """
                    # Extract the downloaded tar.gz file
                    tar -xzf ${FILE_NAME}
                    """
                }
            }
        }

        stage('Push to another Nexus Repository') {
            steps {
                script {
                    sh """
                    # Prepare the artifact for upload to Nexus (assuming Maven is used)
                    mvn deploy:deploy-file \
                    -Dfile=${FILE_NAME} \
                    -DrepositoryId=nexus-repo \
                    -Durl=${NEXUS_UPLOAD_REPO_URL} \
                    -DgroupId=${GROUP_ID} \
                    -DartifactId=${ARTIFACT_ID} \
                    -Dversion=${VERSION} \
                    -Dpackaging=${PACKAGING} \
                    -DgeneratePom=true \
                    -Dusername=${NEXUS_USERNAME} \
                    -Dpassword=${NEXUS_PASSWORD}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "File successfully uploaded to Nexus repository."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}

Explanation of Key Sections:

1. Environment Variables:

NEXUS_DOWNLOAD_REPO_URL: The URL of the Nexus repository from which the .tar.gz file will be downloaded.

NEXUS_UPLOAD_REPO_URL: The URL of the Nexus repository to which the file will be uploaded.

NEXUS_USERNAME and NEXUS_PASSWORD: Use Jenkins' credentials binding to manage sensitive information securely.

FILE_NAME: The name of the .tar.gz file to be downloaded.

GROUP_ID, ARTIFACT_ID, VERSION: Maven artifact details to define the group, artifact, and version.



2. Stage 1 - Download: This stage uses wget to download the .tar.gz file from Nexus with credentials.


3. Stage 2 - Extract: After downloading, the .tar.gz file is extracted using tar.


4. Stage 3 - Push to Nexus: This stage uses Maven's deploy:deploy-file to upload the .tar.gz file to a different Nexus repository.



Adjustments:

Maven: Ensure Maven is installed on the Jenkins agent.

Credentials: Set up Nexus credentials (nexus-username and nexus-password) in Jenkins.

Repository Details: Update repository URLs, group ID, artifact ID, and version details as per your Nexus setup.


Let me know if you need further customization!
