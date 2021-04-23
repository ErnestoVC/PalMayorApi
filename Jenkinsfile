pipeline {
  agent any
  environment {
    dotnet = 'C:\\Program Files\\dotnet\\dotnet.exe'
  }

  stages {
    stage('Checkout') {
      steps {
        git credentialsId: 'netdeployer', url: 'https://github.com/ErnestoVC/PalMayorApi', branch: 'main'
      }
    }
    stage('Restore Packages') {
      steps {
        bat "dotnet restore"
        }
    }
    stage('Clean') {
      steps {
        bat 'dotnet clean'
      }
    }
    stage('Build') {
      steps {
        bat 'dotnet build --configuration Release'
      }
    }
    stage('Pack') {
      steps {
        bat 'dotnet pack --no-build --output nupkgs'
      }
    }
    stage('Running unit tests') {
    steps {
      bat "dotnet add ${workspace}/ApiVP.Tests/ApiVP.Tests.csproj package JUnitTestLogger --version 1.1.0"
      bat "dotnet test ${workspace}/ApiVP.Tests/ApiVP.Tests.csproj --logger \"junit;LogFilePath=\"${WORKSPACE}\"/TestResults/1.0.0.\"${env.BUILD_NUMBER}\"/results.xml\" --configuration release --collect \"Code coverage\""
      powershell '''
      $destinationFolder = \"$env:WORKSPACE/TestResults\"
      if (!(Test-Path -path $destinationFolder)) {New-Item $destinationFolder -Type Directory}
      $file = Get-ChildItem -Path \"$env:WORKSPACE/ApiVP.Tests/ApiVP.Tests/TestResults/*/*.coverage\"
      $file | Rename-Item -NewName testcoverage.coverage
      $renamedFile = Get-ChildItem -Path \"$env:WORKSPACE/ApiVP.Tests/ApiVP.Tests/TestResults/*/*.coverage\"
      Copy-Item $renamedFile -Destination $destinationFolder
      '''
      }        
    }
    stage('Generate report') {
      steps {
        bat "C:\\Users\\netit\\Downloads\\ReportGenerator_4.8.7\netcoreapp3.0\\ReportGenerator.exe -reports:${WORKSPACE}\\TestResults\\xmlresults.coveragexml -targetdir:${WORKSPACE}\\CodeCoverage_${env.BUILD_NUMBER}
      }
    }
    stage('Publish HTML report') {
      steps {
        publishHTML(target: [allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'CodeCoverage_${BUILD_NUMBER}', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: 'Code Coverage Report'])
      }
    }
    stage('Publish'){
      steps{
        bat "dotnet publish ApiVP.Api\\ApiVP.Api.csproj"
      }
    }
  }
}