# CI/CD pipeline: install dependencies, run the unit tests, package the
# function app, and publish the build artifact for deployment.

trigger:
- main

variables:
  # Azure service connection established during pipeline creation.
  azureSubscription: '<your-azure-service-connection>'
  appName: '<your-function-app-name>'
  vmImageName: 'ubuntu-latest'

pool:
  vmImage: ubuntu-latest

steps:
- task: UsePythonVersion@0
  displayName: "Use Python 3.11"
  inputs:
    versionSpec: '3.11'
    architecture: 'x64'

- bash: |
    pip install -r requirements.txt
    pip install pytest
  displayName: "Install dependencies"

- bash: |
    python -m pytest tests/
  displayName: "Run unit tests"

- bash: |
    pip install --target="./.python_packages/lib/site-packages" -r ./requirements.txt
  displayName: "Vendor dependencies for packaging"

- task: ArchiveFiles@2
  displayName: "Archive files"
  inputs:
    rootFolderOrFile: "$(System.DefaultWorkingDirectory)"
    includeRootFolder: false
    archiveFile: "$(System.DefaultWorkingDirectory)/build$(Build.BuildId).zip"

- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(System.DefaultWorkingDirectory)/build$(Build.BuildId).zip'
    artifactName: 'drop'

# To deploy automatically, add an AzureFunctionApp@2 task here referencing
# the azureSubscription and appName variables above.
