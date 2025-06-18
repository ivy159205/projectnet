pipeline {
    agent any

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
                bat 'dotnet publish WebApplication1/WebApplication1/WebApplication1.csproj -c Release -o C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\demo_devops\\publish'
            }
        }

        stage('Setup IIS Sites') {
            steps {
                echo 'Creating IIS websites on port 82 and 83 with separate app pools...'
                powershell '''
                    Import-Module WebAdministration
                    
                    $sites = @(
                        @{ Name='projectnet82'; Port=82; Path='C:\\inetpub\\wwwroot\\projectnet82'; AppPool='projectnet82AppPool' },
                        @{ Name='projectnet83'; Port=83; Path='C:\\inetpub\\wwwroot\\projectnet83'; AppPool='projectnet83AppPool' }
                    )

                    foreach ($site in $sites) {
                        # Create directory if it doesn't exist
                        if (-Not (Test-Path $site.Path)) {
                            New-Item -ItemType Directory -Path $site.Path -Force | Out-Null
                            Write-Host "Created directory: $($site.Path)"
                        }

                        # Create application pool if it doesn't exist
                        try {
                            $pool = Get-WebAppPool -Name $site.AppPool -ErrorAction Stop
                            Write-Host "App pool exists: $($site.AppPool)"
                        }
                        catch {
                            New-WebAppPool -Name $site.AppPool
                            Write-Host "Created app pool: $($site.AppPool)"
                        }

                        # Create website if it doesn't exist
                        try {
                            $website = Get-Website -Name $site.Name -ErrorAction Stop
                            Write-Host "Site exists: $($site.Name)"
                        }
                        catch {
                            New-Website -Name $site.Name -Port $site.Port -PhysicalPath $site.Path -ApplicationPool $site.AppPool
                            Write-Host "Created site: $($site.Name)"
                        }
                    }
                '''
            }
        }

        stage('Stop IIS Application Pools') {
            steps {
                echo 'Stopping IIS Application Pools and Websites...'
                powershell '''
                    Import-Module WebAdministration
                    
                    # Stop websites first
                    $sites = @('projectnet82', 'projectnet83')
                    foreach ($site in $sites) {
                        try {
                            $siteState = Get-WebsiteState -Name $site -ErrorAction Stop
                            if ($siteState.Value -eq 'Started') {
                                Stop-Website -Name $site
                                Write-Host "Stopped website: $site"
                                Start-Sleep -Seconds 2
                            } else {
                                Write-Host "Website $site is already stopped"
                            }
                        }
                        catch {
                            Write-Host "Website $site does not exist or cannot be stopped"
                        }
                    }
                    
                    # Then stop app pools
                    $pools = @('projectnet82AppPool', 'projectnet83AppPool')
                    foreach ($pool in $pools) {
                        try {
                            $state = Get-WebAppPoolState -Name $pool -ErrorAction Stop
                            if ($state.Value -eq 'Started') {
                                Stop-WebAppPool -Name $pool
                                Write-Host "Stopped app pool: $pool"
                                
                                # Wait for app pool to fully stop
                                $timeout = 30
                                $elapsed = 0
                                do {
                                    Start-Sleep -Seconds 1
                                    $elapsed++
                                    $currentState = Get-WebAppPoolState -Name $pool -ErrorAction SilentlyContinue
                                } while ($currentState.Value -eq 'Started' -and $elapsed -lt $timeout)
                                
                                if ($elapsed -ge $timeout) {
                                    Write-Host "Warning: App pool $pool may not have stopped completely"
                                } else {
                                    Write-Host "App pool $pool fully stopped after $elapsed seconds"
                                }
                            } else {
                                Write-Host "App pool $pool is already stopped"
                            }
                        }
                        catch {
                            Write-Host "App pool $pool does not exist or cannot be stopped"
                        }
                    }
                    
                    # Additional cleanup - kill any remaining w3wp processes
                    try {
                        Get-Process -Name "w3wp" -ErrorAction SilentlyContinue | Stop-Process -Force -ErrorAction SilentlyContinue
                        Write-Host "Cleaned up any remaining w3wp processes"
                    }
                    catch {
                        Write-Host "No w3wp processes to clean up"
                    }
                '''
            }
        }

        stage('Copy to IIS folders') {
            parallel {
                stage('Copy to Port 82 folder') {
                    steps {
                        echo 'Copying to Port 82 folder...'
                        retry(3) {
                            powershell '''
                                $source = "publish\\*"
                                $destination = "C:\\inetpub\\wwwroot\\projectnet82\\"
                                
                                # Ensure destination exists
                                if (-Not (Test-Path $destination)) {
                                    New-Item -ItemType Directory -Path $destination -Force | Out-Null
                                }
                                
                                # Use robocopy for better handling of locked files
                                $result = robocopy "publish" "C:\\inetpub\\wwwroot\\projectnet82" /E /R:3 /W:5 /MT:8
                                
                                # Robocopy exit codes: 0-7 are success, 8+ are errors
                                if ($LASTEXITCODE -gt 7) {
                                    throw "Robocopy failed with exit code: $LASTEXITCODE"
                                } else {
                                    Write-Host "Files copied successfully to Port 82 folder"
                                }
                            '''
                        }
                    }
                }
                stage('Copy to Port 83 folder') {
                    steps {
                        echo 'Copying to Port 83 folder...'
                        retry(3) {
                            powershell '''
                                $source = "publish\\*"
                                $destination = "C:\\inetpub\\wwwroot\\projectnet83\\"
                                
                                # Ensure destination exists
                                if (-Not (Test-Path $destination)) {
                                    New-Item -ItemType Directory -Path $destination -Force | Out-Null
                                }
                                
                                # Use robocopy for better handling of locked files
                                $result = robocopy "publish" "C:\\inetpub\\wwwroot\\projectnet83" /E /R:3 /W:5 /MT:8
                                
                                # Robocopy exit codes: 0-7 are success, 8+ are errors
                                if ($LASTEXITCODE -gt 7) {
                                    throw "Robocopy failed with exit code: $LASTEXITCODE"
                                } else {
                                    Write-Host "Files copied successfully to Port 83 folder"
                                }
                            '''
                        }
                    }
                }
            }
        }

        stage('Start IIS Application Pools') {
            steps {
                echo 'Starting IIS Application Pools and Websites...'
                powershell '''
                    Import-Module WebAdministration
                    
                    # Start app pools first
                    $pools = @('projectnet82AppPool', 'projectnet83AppPool')
                    foreach ($pool in $pools) {
                        try {
                            $state = Get-WebAppPoolState -Name $pool -ErrorAction Stop
                            if ($state.Value -ne 'Started') {
                                Start-WebAppPool -Name $pool
                                Write-Host "Started app pool: $pool"
                                
                                # Wait for app pool to fully start
                                $timeout = 30
                                $elapsed = 0
                                do {
                                    Start-Sleep -Seconds 1
                                    $elapsed++
                                    $currentState = Get-WebAppPoolState -Name $pool -ErrorAction SilentlyContinue
                                } while ($currentState.Value -ne 'Started' -and $elapsed -lt $timeout)
                                
                                if ($elapsed -ge $timeout) {
                                    Write-Host "Warning: App pool $pool may not have started completely"
                                } else {
                                    Write-Host "App pool $pool fully started after $elapsed seconds"
                                }
                            } else {
                                Write-Host "App pool $pool is already started"
                            }
                        }
                        catch {
                            Write-Host "Cannot start app pool $pool"
                        }
                    }
                    
                    # Then start websites
                    $sites = @('projectnet82', 'projectnet83')
                    foreach ($site in $sites) {
                        try {
                            $siteState = Get-WebsiteState -Name $site -ErrorAction Stop
                            if ($siteState.Value -ne 'Started') {
                                Start-Website -Name $site
                                Write-Host "Started website: $site"
                            } else {
                                Write-Host "Website $site is already started"
                            }
                        }
                        catch {
                            Write-Host "Cannot start website $site"
                        }
                    }
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'
                script {
                    try {
                        bat 'curl -f http://localhost:82'
                        echo 'Port 82 is responding'
                    } catch (Exception e) {
                        echo 'Port 82 failed: ' + e.getMessage()
                    }
                    
                    try {
                        bat 'curl -f http://localhost:83'
                        echo 'Port 83 is responding'
                    } catch (Exception e) {
                        echo 'Port 83 failed: ' + e.getMessage()
                    }
                }
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
        success {
            echo 'Deployment succeeded!'
        }
    }
}