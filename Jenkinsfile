pipeline {
    agent any

    stages {
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
                bat 'dotnet publish WebApplication1/WebApplication1/WebApplication1.csproj -c Release -o %WORKSPACE%\\publish'
            }
        }

        stage('Stop IIS Application Pools') {
            steps {
                echo 'Stopping IIS Application Pools...'
                script {
                    bat 'powershell -Command "Import-Module WebAdministration; Stop-WebAppPool -Name MyAppPool82 -ErrorAction SilentlyContinue" || echo Warning: Could not stop MyAppPool82'
                    bat 'powershell -Command "Import-Module WebAdministration; Stop-WebAppPool -Name MyAppPool83 -ErrorAction SilentlyContinue" || echo Warning: Could not stop MyAppPool83'
                }
            }
        }

        stage('Copy to IIS folders') {
            parallel {
                stage('Copy to Port 82 folder') {
                    steps {
                        echo 'Copying files to Port 82 IIS folder...'
                        bat 'robocopy "%WORKSPACE%\\publish" "c:\\wwwroot\\myproject82" /E /R:3 /W:10'
                    }
                }
                stage('Copy to Port 83 folder') {
                    steps {
                        echo 'Copying files to Port 83 IIS folder...'
                        bat 'robocopy "%WORKSPACE%\\publish" "c:\\wwwroot\\myproject83" /E /R:3 /W:10'
                    }
                }
            }
        }

        stage('Deploy to IIS - Dual Ports') {
            steps {
                echo 'Deploying to IIS...'
                // Các bước thêm nếu cần
            }
        }

        stage('Start IIS Application Pools') {
            steps {
                echo 'Starting IIS Application Pools...'
                script {
                    bat 'powershell -Command "Import-Module WebAdministration; Start-WebAppPool -Name MyAppPool82 -ErrorAction SilentlyContinue" || echo Warning: Could not start MyAppPool82'
                    bat 'powershell -Command "Import-Module WebAdministration; Start-WebAppPool -Name MyAppPool83 -ErrorAction SilentlyContinue" || echo Warning: Could not start MyAppPool83'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'
                // Ví dụ ping hoặc HTTP request đến localhost:82, :83 nếu cần
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
