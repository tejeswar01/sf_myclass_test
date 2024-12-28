pipeline {
    agent any

    environment {
        SFDX_AUTH_URL = credentials('salesforce-auth-url') // Salesforce Auth URL stored in Jenkins credentials
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning repository...'
                checkout scm
            }
        }

        stage('Setup Salesforce CLI') {
            steps {
                echo 'Authenticating to Salesforce...'
                sh 'sfdx auth:sfdxurl:store --url $SFDX_AUTH_URL --setalias JenkinsOrg --noninteractive'
                sh 'sfdx force:config:set apiVersion=58.0'
            }
        }

        stage('Create Scratch Org') {
            steps {
                echo 'Creating scratch org...'
                sh 'sfdx force:org:create -f config/project-scratch-def.json --setalias ScratchOrg --durationdays 7 --setdefaultusername'
            }
        }

        stage('Deploy Code') {
            steps {
                echo 'Deploying code to scratch org...'
                sh 'sfdx force:source:push'
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running Apex tests...'
                sh 'sfdx force:apex:test:run --resultformat junit --outputdir test-results --codecoverage'
            }
        }

        stage('Archive Results') {
            steps {
                echo 'Archiving test results...'
                junit 'test-results/*.xml'
            }
        }

        stage('Delete Scratch Org') {
            steps {
                echo 'Cleaning up scratch org...'
                sh 'sfdx force:org:delete -u ScratchOrg -p'
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for more details.'
        }
    }
}
