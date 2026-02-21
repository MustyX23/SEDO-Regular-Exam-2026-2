pipeline {
    agent any

    parameters {
        string(name: 'DOTNET_VERSION', defaultValue: '6.0', description: 'Target .NET version')
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    triggers {
        githubPush()
    }

    environment {
        DOTNET_CLI_TELEMETRY_OPTOUT = 'true'
        DOTNET_SKIP_FIRST_TIME_EXPERIENCE = 'true'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    echo "Checking out source code from main branch..."
                    checkout scm
                }
            }
        }

        stage('Restore') {
            steps {
                script {
                    echo "Restoring NuGet packages..."
                    bat 'dotnet restore'
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Building the application..."
                    bat 'dotnet build --configuration Release --no-restore'
                }
            }
        }

        stage('Unit Tests') {
            steps {
                script {
                    echo "Running unit tests..."
                    bat '''
                        dotnet test Homies.Tests/Homies.Tests.csproj ^
                            --configuration Release ^
                            --no-build ^
                            --logger "trx;LogFileName=unit-test-results.trx" ^
                            --collect:"XPlat Code Coverage"
                    '''
                }
            }
        }

        stage('Integration Tests') {
            steps {
                script {
                    echo "Running integration tests..."
                    bat '''
                        dotnet test Homies.IntegrationTests/Homies.IntegrationTests.csproj ^
                            --configuration Release ^
                            --no-build ^
                            --logger "trx;LogFileName=integration-test-results.trx" ^
                            --collect:"XPlat Code Coverage"
                    '''
                }
            }
        }

        stage('Publish Test Results') {
            steps {
                script {
                    echo "Publishing test results..."
                    step([$class: 'MSTestPublisher', testResultsFile: '**/unit-test-results.trx', failOnError: true, keepLongSTDINchars: false])
                    step([$class: 'MSTestPublisher', testResultsFile: '**/integration-test-results.trx', failOnError: true, keepLongSTDINchars: false])
                }
            }
        }
    }

    post {
        always {
            script {
                echo "Pipeline execution completed."
                // Archive test results
                archiveArtifacts artifacts: '**/TestResults/**', allowEmptyArchive: true
            }
        }
        success {
            echo "Build completed successfully!"
        }
        failure {
            echo "Build failed. Please check the logs."
        }
        unstable {
            echo "Build is unstable. Some tests may have failed."
        }
    }
}
