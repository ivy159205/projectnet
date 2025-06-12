pipeline {
    agent any

    stages {
        stage('Clone') {
            steps {
                echo 'Cloning source code...'
                git branch: 'main', url: 'https://github.com/ivy159205/projectnet.git'
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
                bat 'dotnet publish WebApplication1/WebApplication1/WebApplication1.csproj -c Release -o ./publish'
            }
        }

        stage('Copy to IIS folder') {
            steps {
                echo 'Copying files to IIS folder...'
                bat 'xcopy "%WORKSPACE%\\publish" /E /Y /I /R "c:\\wwwroot\\myproject"'
            }
        }

        stage('Deploy to IIS') {
            steps {
                powershell '''
                    Import-Module WebAdministration
                    if (-not (Test-Path IIS:\\Sites\\MySite)) {
                        New-Website -Name "MySite" -Port 81 -PhysicalPath "c:\\wwwroot\\myproject"
                    }
                '''
            }
        }
    }
}
