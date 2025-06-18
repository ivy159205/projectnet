pipeline {
    agent any

    environment {
        PUBLISH_FOLDER = "${WORKSPACE}/publish"
        IIS_APPPOOL = "DefaultAppPool"
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
                bat 'dotnet restore WebApplication1/WebApplication1/WebApplication1.csproj'
            }
        }

        stage('Build') {
            steps {
                echo 'Building the project...'
                bat 'dotnet build WebApplication1/WebApplication1/WebApplication1.csproj --configuration Release'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                bat 'dotnet test WebApplication1/WebApplication1/WebApplication1.csproj --no-build --verbosity normal'
            }
        }

        stage('Publish') {
            steps {
                echo 'Publishing project to folder...'
                bat "dotnet publish WebApplication1/WebApplication1/WebApplication1.csproj -c Release -o ${PUBLISH_FOLDER}"
            }
        }

        stage('Stop IIS Application Pools') {
            steps {
                echo 'Stopping IIS Application Pools...'
                script {
                    bat """
                    powershell -Command "Import-Module WebAdministration;
                    if (Test-Path IIS:\\AppPools\\${env.IIS_APPPOOL}) {
                        Stop-WebAppPool -Name ${env.IIS_APPPOOL} -ErrorAction SilentlyContinue
                    } else {
                        Write-Host 'AppPool ${env.IIS_APPPOOL} does not exist.'
                    }"
                    """
                }
            }
        }

        stage('Copy to IIS folders') {
            parallel {
                stage('Copy to Port 82 folder') {
                    steps {
                        echo 'Copying files to IIS Port 82...'
                        bat "xcopy ${PUBLISH_FOLDER} C:\\inetpub\\wwwroot\\MyWebApp82 /E /Y /I"
                    }
                }
                stage('Copy to Port 83 folder') {
                    steps {
                        echo 'Copying files to IIS Port 83...'
                        bat "xcopy ${PUBLISH_FOLDER} C:\\inetpub\\wwwroot\\MyWebApp83 /E /Y /I"
                    }
                }
            }
        }

        stage('Start IIS Application Pools') {
            steps {
                echo 'Starting IIS Application Pools...'
                script {
                    bat """
                    powershell -Command "Import-Module WebAdministration;
                    if (Test-Path IIS:\\AppPools\\${env.IIS_APPPOOL}) {
                        Start-WebAppPool -Name ${env.IIS_APPPOOL}
                    } else {
                        Write-Host 'AppPool ${env.IIS_APPPOOL} does not exist.'
                    }"
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Deployment complete.'
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
