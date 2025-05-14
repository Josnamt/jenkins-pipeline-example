pipeline {
    parameters {
        booleanParam(name: 'NEW_RUN', defaultValue: true, description: 'Whether to run or not')
        choice(name: 'RUN', choices: ['develop', 'stage', 'prod'], description: 'Selecting environment')
    }

    agent any

    tools {
        jdk 'jdk-17-local'
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
                    if (!params.NEW_RUN) {
                        currentBuild.result = 'ABORTED'
                        error("Pipeline aborted: NEW_RUN is false")
                    }
                    echo "Running environment: ${params.RUN}"
                }
            }
        }

        stage('Build') {
            steps {
                echo "Building with Gradle"
                sh './gradlew clean build -x test'
            }
        }

        stage('Tests') {
            when {
                expression { return params.RUN != 'prod' }
            }
            steps {
                echo "Running tests for ${params.RUN}"
                sh './gradlew test'
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
                echo "Deploying ${APP_NAME} to ${DEPLOY_DIR}"
                script {
                    def jarFile = sh(
                        script: "ls build/libs/*.jar | head -n 1",
                        returnStdout: true
                    ).trim()

                    echo "Found artifact: ${jarFile}"

                    sh """
                        if [ -f "${jarFile}" ]; then
                            cp "${jarFile}" "${DEPLOY_DIR}/"
                            echo "Deployment completed"
                        else
                            echo "File not found: ${jarFile}"
                            exit 1
                        fi
                    """
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
