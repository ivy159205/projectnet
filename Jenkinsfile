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
        
        stage('Stop IIS Application Pools') {
            steps {
                echo 'Stopping IIS Application Pools...'
                script {
                    try {
                        bat 'powershell -Command "Import-Module WebAdministration; Stop-WebAppPool -Name MyAppPool82 -ErrorAction SilentlyContinue"'
                    } catch (Exception e) {
                        echo "Warning: Could not stop MyAppPool82: ${e.getMessage()}"
                    }
                    try {
                        bat 'powershell -Command "Import-Module WebAdministration; Stop-WebAppPool -Name MyAppPool83 -ErrorAction SilentlyContinue"'
                    } catch (Exception e) {
                        echo "Warning: Could not stop MyAppPool83: ${e.getMessage()}"
                    }
                }
            }
        }
        
        stage('Copy to IIS folders') {
            parallel {
                stage('Copy to Port 82 folder') {
                    steps {
                        echo 'Copying files to Port 82 IIS folder...'
                        bat '''
                            robocopy "C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\Demo1\\publish" "c:\\wwwroot\\myproject82" /E /R:3 /W:10
                            if %errorlevel% leq 1 exit 0
                        '''
                    }
                }
                stage('Copy to Port 83 folder') {
                    steps {
                        echo 'Copying files to Port 83 IIS folder...'
                        bat '''
                            robocopy "C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\Demo1\\publish" "c:\\wwwroot\\myproject83" /E /R:3 /W:10
                            if %errorlevel% leq 1 exit 0
                        '''
                    }
                }
            }
        }
        
        stage('Deploy to IIS - Dual Ports') {
            steps {
                powershell '''
                Import-Module WebAdministration
                
                # Tạo Application Pool cho cổng 82 nếu chưa có
                if (-not (Get-IISAppPool -Name "MyAppPool82" -ErrorAction SilentlyContinue)) {
                    New-WebAppPool -Name "MyAppPool82"
                    Set-ItemProperty -Path "IIS:\\AppPools\\MyAppPool82" -Name "processModel.identityType" -Value "ApplicationPoolIdentity"
                }
                
                # Tạo Application Pool cho cổng 83 nếu chưa có
                if (-not (Get-IISAppPool -Name "MyAppPool83" -ErrorAction SilentlyContinue)) {
                    New-WebAppPool -Name "MyAppPool83"
                    Set-ItemProperty -Path "IIS:\\AppPools\\MyAppPool83" -Name "processModel.identityType" -Value "ApplicationPoolIdentity"
                }
                
                # Tạo website cho cổng 82 nếu chưa có
                if (-not (Get-Website -Name "MySite82" -ErrorAction SilentlyContinue)) {
                    New-Website -Name "MySite82" -Port 82 -PhysicalPath "C:\\wwwroot\\myproject82" -ApplicationPool "MyAppPool82"
                    Write-Host "Created website MySite82 on port 82"
                } else {
                    Set-ItemProperty -Path "IIS:\\Sites\\MySite82" -Name "physicalPath" -Value "C:\\wwwroot\\myproject82"
                    Write-Host "Updated MySite82 physical path"
                }
                
                # Tạo website cho cổng 83 nếu chưa có
                if (-not (Get-Website -Name "MySite83" -ErrorAction SilentlyContinue)) {
                    New-Website -Name "MySite83" -Port 83 -PhysicalPath "C:\\wwwroot\\myproject83" -ApplicationPool "MyAppPool83"
                    Write-Host "Created website MySite83 on port 83"
                } else {
                    Set-ItemProperty -Path "IIS:\\Sites\\MySite83" -Name "physicalPath" -Value "C:\\wwwroot\\myproject83"
                    Write-Host "Updated MySite83 physical path"
                }
                
                Write-Host "Both websites configured successfully"
                '''
            }
        }
        
        stage('Start IIS Application Pools') {
            steps {
                echo 'Starting IIS Application Pools...'
                script {
                    try {
                        bat 'timeout /t 5 /nobreak'
                        bat 'powershell -Command "Import-Module WebAdministration; Start-WebAppPool -Name MyAppPool82"'
                        bat 'powershell -Command "Import-Module WebAdministration; Start-WebAppPool -Name MyAppPool83"'
                    } catch (Exception e) {
                        echo "Warning: Could not start application pools: ${e.getMessage()}"
                        echo "Attempting IIS reset as fallback..."
                        bat 'iisreset /restart'
                    }
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'
                powershell '''
                Import-Module WebAdministration
                
                # Kiểm tra trạng thái websites
                $site82 = Get-Website -Name "MySite82" -ErrorAction SilentlyContinue
                $site83 = Get-Website -Name "MySite83" -ErrorAction SilentlyContinue
                
                if ($site82) {
                    Write-Host "MySite82 Status: $($site82.State) - Port: 82"
                }
                if ($site83) {
                    Write-Host "MySite83 Status: $($site83.State) - Port: 83"
                }
                
                # Kiểm tra Application Pools
                $pool82 = Get-IISAppPool -Name "MyAppPool82" -ErrorAction SilentlyContinue
                $pool83 = Get-IISAppPool -Name "MyAppPool83" -ErrorAction SilentlyContinue
                
                if ($pool82) {
                    Write-Host "MyAppPool82 Status: $($pool82.State)"
                }
                if ($pool83) {
                    Write-Host "MyAppPool83 Status: $($pool83.State)"
                }
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Deployment completed successfully!'
            echo 'Application is available at:'
            echo '- http://localhost:82 (MySite82)'
            echo '- http://localhost:83 (MySite83)'
        }
        failure {
            echo 'Deployment failed!'
        }
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}