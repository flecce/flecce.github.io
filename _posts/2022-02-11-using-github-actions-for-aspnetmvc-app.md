# Using Github Actions for ASP.Net MVC
<code>English</code> <code>Tech</code>

## Intro

This post shows you how to create a build/release pipeline for a ASP.Net MVC (.NET Framework) application.

### Create a simple app

![New MVC project](/images/img_1.png "New MVC project")

### Create publish profile

To make this simple I've created a Visual Studio Folder profile:

![New MVC project](/images/img_2.png "New MVC project")

### Create a new a GitHub Actions

```yaml
name: MVC App

on:
  push:
    branches: [ master ]

jobs:
  build:
    runs-on: windows-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v2
        
      - name: Setup MSBuild
        uses: microsoft/setup-msbuild@v1
      
      - name: Setup NuGet
        uses: NuGet/setup-nuget@v1.0.2
        
      - name: Restore Packages
        run: nuget restore .\app.sln
        
      - name: Build and Publish Web App
        run: |
          msbuild .\app.sln /p:Configuration=Release /p:DeployOnBuild=true /p:PublishProfile=FolderProfile

      - name: Upload Artifact
        uses: actions/upload-artifact@v2
        with:
          name: artifacts
          retention-days: 1

  release:
    needs: [ build ]
    runs-on: web
    
    steps:
      - name: Download a single artifact
        uses: actions/download-artifact@v2
        with:
          name: artifacts
```

The pipeline has two jobs: a build and release job. It starts when a push is made on master branch.
The build job runs self-hosted runner on staging server.

The first step of build job is the git checkout, if you do not specify ref parameter the checkout is on default branch (in my case is master branch).
After checkout the pipeline setup MSBuild, restore nuget packages and publish the app (with previous created folder profile).

The last step is the publish of the artifacts (you can add path property to specify which files you want to upload).

The release job starts when the build step finish succesfully. The only step is the downloading artifacts and with path
property (not in this example) you can specify the download path.
