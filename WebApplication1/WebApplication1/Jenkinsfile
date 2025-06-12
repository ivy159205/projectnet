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
                bat 'dotnet restore'
            }
        }

        stage('Build') {
            steps {
                echo 'Building the project...'
                bat 'dotnet build --configuration Release'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                bat 'dotnet test --no-build --verbosity normal'
            }
        }

        stage('Publish') {
            steps {
                echo 'Publishing project to /publish...'
                bat 'dotnet publish -c Release -o ./publish'
            }
        }

        stage('Copy to IIS folder') {
            steps {
                echo 'Copying published files to IIS folder...'
                bat 'xcopy "%WORKSPACE%\\publish" "C:\\wwwroot\\myproject" /E /Y /I /R'
            }
        }

        stage('Deploy to IIS') {
            steps {
                echo 'Deploying to IIS...'
                powershell '''
                    Import-Module WebAdministration
                    if (-not (Test-Path IIS:\\Sites\\MySite)) {
                        New-Website -Name "MySite" -Port 81 -PhysicalPath "C:\\wwwroot\\myproject"
                    }
                '''
            }
        }
    }
}
