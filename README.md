# Flutter CI/CD pipeline template

**Note:** still in progress...

[![Build Status](https://dev.azure.com/florisdevreese/Flutter-CI-CD/_apis/build/status/FlorisDevreese.Flutter-CI-CD?branchName=master)](https://dev.azure.com/florisdevreese/Flutter-CI-CD/_build/latest?definitionId=8&branchName=master)

This project has the following goals:
- Setup a CI/CD pipeline for a flutter app that builds an Android and an IOS app
- Show that you can build the IOS app without a physical macOS device
- Provide inspiration/templates for setting up CI/CD pipelines for your flutter project

This project uses the following technologies:
- Azure DevOps for setting up the pipelines
- Google Firebase to do the app distribution to the testers

Everything is encapsulated in one [azure-pipelines.yml](./azure-pipelines.yml) file. What follows is a chronical explenation of all the steps I had to do to create the whole pipeline.

## 1. Setting up Android CI Build

1. Create an Azure DevOps project
2. Install this [Azure DevOps Flutter extension](https://marketplace.visualstudio.com/items?itemName=aloisdeniel.flutter) on my Azure DevOps project
3. Create a simple new Azure DevOps build pipeline for the android build. This pipeline doens't yet include signing
    - See [this the early version of the pipeline](https://github.com/FlorisDevreese/Flutter-CI-CD/commit/5361300046d5537c8f279a73b10eb3bf3b7f2aab)
    - **Note:** I encounter [this issue](#1.-Flutter-installation-fails-on-Ubuntu-agent)
4. Add signing to the android build
    - Create a keystore. See [here how](https://flutter.dev/docs/deployment/android#create-a-keystore)
    - add the keystore file to secureFiles in Azure DevOps (I called the keystore file `releaseKeyStore.jks`)
    - adjust the pipeline ([see all changes here](https://github.com/FlorisDevreese/Flutter-CI-CD/commit/35a112ab3b4572d06cdbb28bd61399a5e4a33434))
        - Add variables to the pipeline.
            - **Note:** the passwords are saved as secret variables
        - Add a step to download the `releaseKeyStore.jks` secure file
        - Add step to set environment variables. These environment variables are used in the signing process
    - Edit build.gradle file to include signing when release building.
        - [see the edits here](https://github.com/FlorisDevreese/Flutter-CI-CD/commit/9233630cd5761185a4c2849e5e46b1b2621c4175)



## Problems I encountered

Here's a list of all the inconvencies I encounter while setting up this project

### 1. Flutter installation fails on Ubuntu agent
[This error](https://dev.azure.com/florisdevreese/Flutter-CI-CD/_build/results?buildId=177&view=logs&j=d8c010d3-91c2-546a-b1cf-ee2cd3c34608&t=fc1891f0-4456-57de-7f19-2aff778a973a&l=16) occurs when you try to install flutter on a Ubuntu agent in your pipeline.
```log
Downloading: https://storage.googleapis.com/flutter_infra/releases/stable/linux/flutter_linux_v1.12.13+hotfix.9-stable.zip
##[error]Error: Unexpected HTTP response: 404
```
This is a [bug](https://github.com/aloisdeniel/vsts-flutter-tasks/issues/14) in the Flutter extension. A work around is running the build on a windows, or macOS agent instead of on an ubuntu agent. [See workaround here](https://github.com/FlorisDevreese/Flutter-CI-CD/commit/89b9ce3d52879babb0bf9b62fac1021fb6feda49).

**Note:** this bug was encountered on 26/04/2020.