pipeline {
    agent any

    environment {
        BUILD_CONFIGURATION = 'Release'
        PUBLISH_DIR = "${WORKSPACE}/publish"
        PROJECT_PATH = 'WebApplication1/WebApplication1/WebApplication1.csproj'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Restore packages') {
            steps {
                echo 'Restoring NuGet packages...'
                bat "dotnet restore ${PROJECT_PATH}"
            }
        }

        stage('Build') {
            steps {
                echo 'Building the project...'
                bat "dotnet build ${PROJECT_PATH} --configuration ${BUILD_CONFIGURATION}"
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                bat "dotnet test ${PROJECT_PATH} --no-build --verbosity normal"
            }
        }

        stage('Publish') {
            steps {
                echo 'Publishing project to folder...'
                bat "dotnet publish ${PROJECT_PATH} -c ${BUILD_CONFIGURATION} -o ${PUBLISH_DIR}"
            }
        }

        stage('Stop IIS Application Pools') {
            steps {
                echo 'Stopping IIS Application Pools...'
                bat '''
                    powershell -NoProfile -Command "Import-Module WebAdministration; `
                    if (Test-Path 'IIS:\\AppPools\\DefaultAppPool') { `
                        Stop-WebAppPool -Name 'DefaultAppPool' -ErrorAction SilentlyContinue `
                    } else { `
                        Write-Host 'AppPool DefaultAppPool does not exist.' `
                    }"
                '''
            }
        }

        stage('Copy to IIS folders') {
            parallel {
                stage('Copy to Port 82 folder') {
                    steps {
                        bat "xcopy /E /Y /I ${PUBLISH_DIR} C:\\inetpub\\port82\\"
                    }
                }
                stage('Copy to Port 83 folder') {
                    steps {
                        bat "xcopy /E /Y /I ${PUBLISH_DIR} C:\\inetpub\\port83\\"
                    }
                }
            }
        }

        stage('Start IIS Application Pools') {
            steps {
                echo 'Starting IIS Application Pools...'
                bat '''
                    powershell -NoProfile -Command "Import-Module WebAdministration; `
                    if (Test-Path 'IIS:\\AppPools\\DefaultAppPool') { `
                        Start-WebAppPool -Name 'DefaultAppPool' `
                    } else { `
                        Write-Host 'AppPool DefaultAppPool does not exist.' `
                    }"
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Deployment complete. You may verify on browser.'
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
