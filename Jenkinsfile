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
                bat '''
        powershell -NoProfile -ExecutionPolicy Bypass -Command "
        Import-Module WebAdministration;

        $sites = @(
            @{ Name='projectnet82'; Port=82; Path='C:\\inetpub\\wwwroot\\projectnet82'; AppPool='projectnet82AppPool' },
            @{ Name='projectnet83'; Port=83; Path='C:\\inetpub\\wwwroot\\projectnet83'; AppPool='projectnet83AppPool' }
        );

        foreach ($site in $sites) {
            if (-Not (Test-Path $site.Path)) {
                New-Item -ItemType Directory -Path $site.Path | Out-Null
            }

            if (-Not (Get-WebAppPoolState -Name $site.AppPool -ErrorAction SilentlyContinue)) {
                New-WebAppPool -Name $site.AppPool;
                Write-Host 'Created app pool: ' $site.AppPool
            } else {
                Write-Host 'App pool exists: ' $site.AppPool
            }

            if (-Not (Get-Website -Name $site.Name -ErrorAction SilentlyContinue)) {
                New-Website -Name $site.Name -Port $site.Port -PhysicalPath $site.Path -ApplicationPool $site.AppPool;
                Write-Host 'Created site: ' $site.Name
            } else {
                Write-Host 'Site exists: ' $site.Name
            }
        }"
        '''
            }
        }

        stage('Stop IIS Application Pools') {
            steps {
                echo 'Stopping IIS Application Pools...'
                bat 'powershell -NoProfile -Command "Import-Module WebAdministration; if (Test-Path \\"IIS:\\\\AppPools\\\\DefaultAppPool\\") { Stop-WebAppPool -Name \\"DefaultAppPool\\" -ErrorAction SilentlyContinue } else { Write-Host \\"AppPool DefaultAppPool does not exist.\\" }"'
            }
        }

        stage('Copy to IIS folders') {
            parallel {
                stage('Copy to Port 82 folder') {
                    steps {
                        echo 'Copying to Port 82 folder...'
                        bat 'xcopy /E /Y publish\\* C:\\inetpub\\wwwroot\\projectnet82\\'
                    }
                }
                stage('Copy to Port 83 folder') {
                    steps {
                        echo 'Copying to Port 83 folder...'
                        bat 'xcopy /E /Y publish\\* C:\\inetpub\\wwwroot\\projectnet83\\'
                    }
                }
            }
        }

        stage('Start IIS Application Pools') {
            steps {
                echo 'Starting IIS Application Pools...'
                bat 'powershell -NoProfile -Command "Import-Module WebAdministration; Start-WebAppPool -Name \\"DefaultAppPool\\""'
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'
                bat 'curl http://localhost:82 || curl http://localhost:83'
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
