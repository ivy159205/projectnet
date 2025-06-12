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
                        bat 'powershell -Command "Import-Module WebAdministration; Stop-WebAppPool -Name DefaultAppPool"'
                    } catch (Exception e) {
                        echo "Warning: Could not stop DefaultAppPool: ${e.getMessage()}"
                    }
                    try {
                        bat 'powershell -Command "Import-Module WebAdministration; Stop-WebAppPool -Name MyAppPool82 -ErrorAction SilentlyContinue"'
                    } catch (Exception e) {
                        echo "Warning: Could not stop MyAppPool82: ${e.getMessage()}"
                    }
                }
            }
        }
        
        stage('Copy to IIS folders') {
            parallel {
                stage('Copy to Port 81 folder') {
                    steps {
                        echo 'Copying files to Port 81 IIS folder...'
                        bat '''
                            robocopy "C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\Demo1\\publish" "c:\\wwwroot\\myproject81" /E /R:3 /W:10
                            if %errorlevel% leq 1 exit 0
                        '''
                    }
                }
                stage('Copy to Port 82 folder') {
                    steps {
                        echo 'Copying files to Port 82 IIS folder...'
                        bat '''
                            robocopy "C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\Demo1\\publish" "c:\\wwwroot\\myproject82" /E /R:3 /W:10
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
                
                # Tạo website cho cổng 81 nếu chưa có
                if (-not (Get-Website -Name "MySite81" -ErrorAction SilentlyContinue)) {
                    New-Website -Name "MySite81" -Port 81 -PhysicalPath "C:\\wwwroot\\myproject81" -ApplicationPool "DefaultAppPool"
                    Write-Host "Created website MySite81 on port 81"
                } else {
                    Set-ItemProperty -Path "IIS:\\Sites\\MySite81" -Name "physicalPath" -Value "C:\\wwwroot\\myproject81"
                    Write-Host "Updated MySite81 physical path"
                }
                
                # Tạo website cho cổng 82 nếu chưa có
                if (-not (Get-Website -Name "MySite82" -ErrorAction SilentlyContinue)) {
                    New-Website -Name "MySite82" -Port 82 -PhysicalPath "C:\\wwwroot\\myproject82" -ApplicationPool "MyAppPool82"
                    Write-Host "Created website MySite82 on port 82"
                } else {
                    Set-ItemProperty -Path "IIS:\\Sites\\MySite82" -Name "physicalPath" -Value "C:\\wwwroot\\myproject82"
                    Write-Host "Updated MySite82 physical path"
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
                        bat 'powershell -Command "Import-Module WebAdministration; Start-WebAppPool -Name DefaultAppPool"'
                        bat 'powershell -Command "Import-Module WebAdministration; Start-WebAppPool -Name MyAppPool82"'
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
                $site81 = Get-Website -Name "MySite81" -ErrorAction SilentlyContinue
                $site82 = Get-Website -Name "MySite82" -ErrorAction SilentlyContinue
                
                if ($site81) {
                    Write-Host "MySite81 Status: $($site81.State) - Port: 81"
                }
                if ($site82) {
                    Write-Host "MySite82 Status: $($site82.State) - Port: 82"
                }
                
                # Kiểm tra Application Pools
                $pool1 = Get-IISAppPool -Name "DefaultAppPool" -ErrorAction SilentlyContinue
                $pool2 = Get-IISAppPool -Name "MyAppPool82" -ErrorAction SilentlyContinue
                
                if ($pool1) {
                    Write-Host "DefaultAppPool Status: $($pool1.State)"
                }
                if ($pool2) {
                    Write-Host "MyAppPool82 Status: $($pool2.State)"
                }
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Deployment completed successfully!'
            echo 'Application is available at:'
            echo '- http://localhost:81 (MySite81)'
            echo '- http://localhost:82 (MySite82)'
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