# Docs for the Azure Web Apps Deploy action: https://github.com/Azure/webapps-deploy
# More GitHub Actions for Azure: https://github.com/Azure/actions

name: Build and deploy ASP app to Azure Web App - hapinessdotworld

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  build:
    runs-on: windows-latest

    steps:
      - uses: actions/checkout@v2

      - name: Setup MSBuild path
        uses: microsoft/setup-msbuild@v1.0.2

      - name: Setup NuGet
        uses: NuGet/setup-nuget@v1.0.5

      - name: Restore NuGet packages
        run: nuget restore

      - name: Publish to folder
        run: msbuild /nologo /verbosity:m /t:Build /t:pipelinePreDeployCopyAllFilesToOneFolder /p:_PackageTempDir="\published\"

      - name: Upload artifact for deployment job
        uses: actions/upload-artifact@v2
        with:
          name: ASP-app
          path: '/published/**'

  deploy:
    runs-on: windows-latest
    needs: build
    environment:
      name: 'production'
      url: ${{ steps.deploy-to-webapp.outputs.webapp-url }}

    steps:
      - name: Download artifact from build job
        uses: actions/download-artifact@v2
        with:
          name: ASP-app

      - name: Deploy to Azure Web App
        id: deploy-to-webapp
        uses: azure/webapps-deploy@v2
        with:
          app-name: 'hapinessdotworld'
          slot-name: 'production'
          publish-profile: ${{ secrets.AzureAppService_PublishProfile_1234 }}
          package: .
