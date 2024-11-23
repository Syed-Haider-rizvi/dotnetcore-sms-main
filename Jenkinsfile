pipeline {
    agent any

    environment {
        DOTNET_CLI_HOME = "C:\\Program Files\\dotnet"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    // Restoring dependencies
                    bat "dotnet restore"

                    // Building the application
                    bat "dotnet build --configuration Release"
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Running tests
                    bat "dotnet test --no-restore --configuration Release"
                }
            }
        }

        stage('Publish') {
            steps {
                script {
                    // Publishing the application
                    bat "dotnet publish ./webapp/webapp.csproj --no-restore --configuration Release --output ./publish"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Using withCredentials to inject credentials
                    withCredentials([usernamePassword(credentialsId: 'coreuser', passwordVariable: 'CREDENTIAL_PASSWORD', usernameVariable: 'CREDENTIAL_USERNAME')]) {
                        powershell '''
                        # Create a PSCredential object from the environment variables for username and password
                        $credentials = New-Object System.Management.Automation.PSCredential($env:CREDENTIAL_USERNAME, (ConvertTo-SecureString $env:CREDENTIAL_PASSWORD -AsPlainText -Force))

                        # Create a new drive 'X' pointing to the remote share with the specified credentials
                        New-PSDrive -Name X -PSProvider FileSystem -Root "\\\\WIN-LASFB11DPMP\\coreapp3" -Persist -Credential $credentials

                        # Copy files from the publish directory to the remote location
                        Copy-Item -Path './publish/*' -Destination 'X:' -Force

                        # Clean up by removing the drive
                        Remove-PSDrive -Name X
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Build, test, and publish successful!'
        }
        failure {
            echo 'There was an error during the build or deployment process.'
        }
    }
}
