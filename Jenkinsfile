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
                bat 'dotnet publish WebApplication1/WebApplication1/WebApplication1.csproj -c Release -o ./publish'
            }
        }
        
        stage('Stop IIS Application Pool') {
            steps {
                echo 'Stopping IIS Application Pool...'
                bat 'powershell -Command "Import-Module WebAdministration; Stop-WebAppPool -Name \'DefaultAppPool\'"'
            }
        }
        
        stage('Copy to IIS folder') {
            steps {
                echo 'Copying files to IIS folder...'
                bat '''
                    robocopy "C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\Demo1\\publish" "c:\\wwwroot\\myproject" /E /R:3 /W:10
                    if %errorlevel% leq 1 exit 0
                '''
            }
        }
        
        stage('Start IIS Application Pool') {
            steps {
                echo 'Starting IIS Application Pool...'
                bat 'powershell -Command "Import-Module WebAdministration; Start-WebAppPool -Name \'DefaultAppPool\'"'
            }
        }
        
        stage('Deploy to IIS') {
            steps {
                echo 'Deployment completed successfully!'
            }
        }
    }
}