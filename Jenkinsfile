@Library('jenk_lib') _

pipeline {
    parameters {
        booleanParam(name: 'NEW_RUN', defaultValue: true, description: 'Whether to run or not')
        choice(name: 'RUN', choices: ['develop', 'stage', 'prod'], description: 'Selecting environment')
    }

    agent any

    tools {
        jdk 'jdk-21-local'
    }

    environment {
        APP_NAME = 'SampleApp'
        DEPLOY_DIR = '/opt/deploy/'
    }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '4'))
        disableConcurrentBuilds()
        timeout(time: 1, unit: 'HOURS')
    }

    stages {
        stage('Initialize') {
            steps {
                script {
                    pipelineStages.initializePipeline()
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    pipelineStages.buildApp()
                }
            }
        }

        stage('Tests') {
            when {
                expression { return params.RUN != 'prod' }
            }
            steps {
                script {
                    pipelineStages.runTests()
                }
            }
        }

        stage('Deploy') {
            when {
                allOf {
                    branch 'develop'  
                    environment name: 'RUN', value: 'prod'
                }
            }
            steps {
                
                script {
                    pipelineStages.deployApp()
                }
            }
        }
    }

    post {
        always {
            echo "Clean workspace"
            deleteDir()
        }
        success {
            echo "Pipeline completed successfully"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
