pipeline {
    agent any
    environment {
        TOWER_SERVER = 'your-tower-server' // Name of the Ansible Tower server configured in Jenkins
        JOB_TEMPLATE = 'your-job-template' // Name of the job template in Ansible Tower
    }
    stages {
        stage('Trigger Ansible Tower Job') {
            steps {
                script {
                    def towerJob = ansibleTower(
                        towerServer: "${TOWER_SERVER}",
                        templateType: 'job',
                        jobTemplate: "${JOB_TEMPLATE}",
                        importTowerLogs: true,
                        removeColor: true,
                        credential: 'your-credential-id', // ID of the credential configured in Jenkins
                        extraVars: [
                            build_number: "${env.BUILD_NUMBER}"
                        ]
                    )
                    echo "Triggered Ansible Tower job with build number: ${env.BUILD_NUMBER}"
                }
            }
        }
    }
}
