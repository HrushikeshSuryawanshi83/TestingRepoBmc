pipeline {
    agent any

    stages {

        stage('Checkout Friend Repo') {
            steps {
                checkout scm
            }
        }

        stage('Download SLA Analyzer') {
            steps {

                bat 'git clone https://github.com/HrushikeshSuryawanshi83/jenkisforbmc.git analyzer'

            }
        }

        stage('Install Node Packages') {
            steps {

                dir('analyzer') {
                    bat 'npm ci'
                }

            }
        }

        stage('Install Python Packages') {
            steps {

                dir('analyzer') {
                    bat 'pip install -r requirements.txt'
                }

            }
        }

        stage('Detect COBOL Files') {

            steps {

                script {

                    def diffRaw = bat(
                        script: 'git diff --name-only HEAD~1 HEAD',
                        returnStdout: true
                    ).trim()

                    echo "Changed Files:\n${diffRaw}"

                    def diff = diffRaw ? diffRaw.split('\n') : []

                    def cobol = diff.findAll { file ->
                        file.toLowerCase().endsWith('.cbl') ||
                        file.toLowerCase().endsWith('.cob')
                    }

                    env.COBOL_FILES = cobol.join(' ')

                    if (env.COBOL_FILES?.trim()) {
                        echo "Detected COBOL Files: ${env.COBOL_FILES}"
                    } else {
                        echo "No COBOL files changed."
                    }
                }
            }
        }

        stage('Run SLA Analysis') {

            when {
                expression { env.COBOL_FILES?.trim() }
            }

            steps {

                echo "Running SLA Analysis..."

                bat returnStatus: true,
                script: "node analyzer/ci/runAnalysis.js ${env.COBOL_FILES}"

            }
        }
    }

    post {

        success {
            echo "Pipeline completed successfully."
        }

        always {
            echo "Build finished."
        }
    }
}