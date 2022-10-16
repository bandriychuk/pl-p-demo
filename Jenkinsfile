#!/usr/bin/env groovy

final boolean IS_MAIN_BRANCH = env.BRANCH_IS_PRIMARY == 'true'
final String CRON_PATTERN = IS_MAIN_BRANCH ? "30 5 * * *" : ""

if (!IS_MAIN_BRANCH) stopOlderBuilds()

pipeline {
    agent { label 'standard' }

    triggers {
        cron(CRON_PATTERN)
    }

    options {
        // keep the 10 most recent builds that are at most 30 days old
        buildDiscarder(logRotator(numToKeepStr: '10', daysToKeepStr: '30'))
        timeout(time: 120, unit: 'MINUTES')
        timestamps()
        parallelsAlwaysFailFast()
    }

    stages {
        stage('UI tests') {
            steps {
                sh """
                    docker-compose  -p ${env.JOB_NAME} \\
                    -f docker-compose.jenkins.yaml run -T \\
                    java ./gradlew clean ui-tests:smokeSuite \\
                    --info
                   """
            }
        }
    }

    post {
        always {
            script {
                allure([
                        results: [[path: "ui-tests/build/allure-results"], [path: "api-tests/build/allure-results"]]
                ])
            }
            junit allowEmptyResults: true, testResults: '**/build/test-results/*/*.xml'

            sh """
              docker-compose -p ${env.JOB_NAME} \\
              --file docker-compose.jenkins.yaml \\
              down --volumes --remove-orphans --rmi local
            """
        }
    }
}