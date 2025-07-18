pipeline {
    agent any
    environment {
        DOTNET_ROOT = "C:\Program Files\dotnet"
        MSDEPLOY_PATH = "C:\Program Files\IIS\Microsoft Web Deploy V3\\msdeploy.exe"
        SITE_NAME = "SampleApi"
        PACKAGE_DIR = "publish"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/hitesh-sk/Dotnet-project.git'
            }
        }
        stage('Build & Test') {
            steps {
                bat """
                dotnet restore
                dotnet build --configuration Release
                dotnet test --configuration Release
                """
            }
        }
        stage('Publish') {
            steps {
                bat 'dotnet publish -c Release -o %PACKAGE_DIR%'
            }
        }
        stage('Deploy to IIS') {
            steps {
                bat """
                "%MSDEPLOY_PATH%" -verb:sync -source:contentPath="%PACKAGE_DIR%" -dest:auto,computerName="https://server:8172/msdeploy.axd?site=%SITE_NAME%",userName="admin",password="admin",authType="basic" -allowUntrusted
                """
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials-id') {
                        def image = docker.build("hitace/dotnetapp:${env.BUILD_NUMBER}")
                        image.push()
                    }
                }
            }
        }
    }
    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Deployment failed!"
        }
    }
}
