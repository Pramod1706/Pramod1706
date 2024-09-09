pipeline {
    agent any

    parameters {
        // Simulate checkmarks using boolean parameters
        booleanParam(name: 'RUN_BUILD', defaultValue: true, description: 'Run Build Stage')
        booleanParam(name: 'RUN_TEST', defaultValue: true, description: 'Run Test Stage')
        booleanParam(name: 'RUN_DEPLOY', defaultValue: true, description: 'Run Deploy Stage')
    }

    stages {
        stage('Build') {
            when {
                expression { return params.RUN_BUILD }
            }
            steps {
                echo 'Running Build Stage...'
                // Add your build steps here
            }
        }

        stage('Test') {
            when {
                expression { return params.RUN_TEST }
            }
            steps {
                echo 'Running Test Stage...'
                // Add your test steps here
            }
        }

        stage('Deploy') {
            when {
                expression { return params.RUN_DEPLOY }
            }
            steps {
                echo 'Running Deploy Stage...'
                // Add your deploy steps here
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline completed.'
        }
    }
}
