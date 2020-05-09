# Flutter CI/CD pipeline template

**Note:** Continious Delivery has not been integrated yet into the pipeline. 😏

[![Build Status](https://dev.azure.com/florisdevreese/Flutter-CI-CD/_apis/build/status/FlorisDevreese.Flutter-CI-CD?branchName=master)](https://dev.azure.com/florisdevreese/Flutter-CI-CD/_build/latest?definitionId=8&branchName=master)

This project has the following goals:
- Setup a CI/CD pipeline for a flutter app that builds an Android and an IOS app
- Show that you can build the IOS app **without** a physical macOS device
- Provide inspiration/templates for setting up CI/CD pipelines for your flutter project
- Show how to link your app to Firebase

This project uses the following technologies:
- Azure DevOps for setting up the pipelines
- Google Firebase to do the app distribution to the testers

This project starts from the basic *create new Flutter project* project. There is no extra functionallity added to the app. This only thing added to this project is a CI/CD pipeline for the basic Flutter app.

Both the Android pipeline and the IOS pipeline are encapsulated into one [`azure-pipelines.yml`](./azure-pipelines.yml) file. What follows is a chronological explenation of all the steps I undertook to create the whole pipeline.

## 1. Setting up Android CI build

1. Create an Azure DevOps project
2. Install this [Azure DevOps Flutter extension](https://marketplace.visualstudio.com/items?itemName=aloisdeniel.flutter) on my Azure DevOps project
3. Create a simple new Azure DevOps build pipeline for the android build. This pipeline doens't yet include signing
    - See [this the early version of the pipeline](https://github.com/FlorisDevreese/Flutter-CI-CD/commit/5361300046d5537c8f279a73b10eb3bf3b7f2aab)
4. Add signing to the android build
    - Create a keystore. See [here how](https://flutter.dev/docs/deployment/android#create-a-keystore)
    - Add the keystore file to [Azure DevOps Secure Files](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/secure-files?view=azure-devops) library (I called the keystore file `releaseKeyStore.jks`)
    - Adjust the pipeline ([see changes here](https://github.com/FlorisDevreese/Flutter-CI-CD/commit/35a112ab3b4572d06cdbb28bd61399a5e4a33434))
        - Add variables to the pipeline.
            - **Note:** the passwords are saved as secret variables
        - Add a step to download the `releaseKeyStore.jks` secure file
        - Add step to set environment variables. These environment variables are used in the signing process
    - Edit build.gradle file to include signing when release building.
        - [See changes here](https://github.com/FlorisDevreese/Flutter-CI-CD/commit/9233630cd5761185a4c2849e5e46b1b2621c4175)

## 2. Setting up the IOS CI build

1. Add a job to the pipeline for IOS building. [See changes here](https://github.com/FlorisDevreese/Flutter-CI-CD/commit/9a75250edecaacc978651ab793ec50bbe0a762a9)
2. Add certificate and provisioning profile to the CI build
    - [Enroll in the Apple developer program](https://developer.apple.com/programs/enroll/)
    - In the [Apple developer console](https://developer.apple.com/account/resources/certificates/list) create a new Certificate
        - Choose `Apple Distribution` as the type of the certificate
        - When asked to upload a `Certificate Signing Request`, create a new certificate by running these commands in Windows, and upload the generated `certificateSigningRequest.certSigningRequest`
            ```powershell
            openssl genrsa -out mykey.key 2048
            openssl req -new -key mykey.key -out certificateSigningRequest.certSigningRequest  -subj "/emailAddress=yourAddress@example.com, CN=Your Name, C=BE"
            ```
        - Download the generated `distribution.cer` Certificate from the Apple developer console
        - Convert the `distribution.cer` file into a `.p12` file by running these commands on Windows
            ```powershell
            # convert .cer to .pem
            openssl x509 -in distribution.cer -inform DER -out distribution.pem -outform PEM
            # combine the .pem and the .key file into one .p12 file
            openssl pkcs12 -export -out distribution.p12 -inkey mykey.key -in distribution.pem
            ```
        - Upload `distribution.p12` to your [Azure DevOps Secure Files](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/secure-files?view=azure-devops) library
    - Register a new Identifier in [Apple developer console](https://developer.apple.com/account/resources/identifiers/list)
        - Choose `App IDs` as the type of the identifier
        - Enter your package ID. In this example it is `com.example.ciCdTemplate`
        - Enter a description for the identifier. I chose `Flutter CI CD`
        - No need to edit the capablities
        - Register the Identifier
    - In [Apple developer console](https://developer.apple.com/account/resources/profiles/list) create a new profile
        - Choose `Ad Hoc` Distribution type
        - Next choose the App ID you registered. In chose `Flutter CI CD`
        - Next choose the Certificate you previously created.
        - Next choose a device for which this profile will work.
            - Yes you'll have to register an iPhone device in the [Apple developer portal](https://developer.apple.com/account/resources/devices/list). You can use a friends iphone for this
        - Give the Provisioning profile a name. I chose `Flutter CI CD - ad hoc`
        - Download the generated provisioning profile. For me the file is called `Flutter_CI_CD__ad_hoc.mobileprovision`
        - Upload the `Flutter_CI_CD__ad_hoc.mobileprovision` file to your [Azure DevOps Secure Files](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/secure-files?view=azure-devops) library
    - Edit the pipeline ([see changes here](https://github.com/FlorisDevreese/Flutter-CI-CD/commit/b8a5b9e15516b594374aad23c213ef4ddd61f465))
        - Add `InstallAppleCertificate` and `InstallAppleProvisioningProfile` steps to the pipeline
            - Be sure to add the password used to create the `distribution.p12` to the variables. I call the variable `distribution.p12-password`
3. Add signing and packaging to `.ipa` ([see changes here](https://github.com/FlorisDevreese/Flutter-CI-CD/commit/e7d6299c74ec40b5fa37508fc6d6d542673b2431))
    - Add [`exportOptions.plist`](./ios/exportOptions.plist)
        - Use your own teamID instead of mine. You can find your teamID in the [Apple developer console](https://developer.apple.com/account/resources/certificates/list) in the top right corner
        - Use your own packageID instead of my `com.example.ciCdTemplate`
        - Use your own provisioning profile name instead of my `Flutter CI CD - ad hoc`
    - Add `XCode` step to pipeline
    - Add `PublishBuildArtifacts` step to pipeline

## 3. Link app with Firebase

Do this steps to link your app with [Firebase](https://firebase.google.com/). All code changes for these steps are visible in [this commit](https://github.com/FlorisDevreese/Flutter-CI-CD/commit/27fe199ec4e5d5c6fc196cab2816d9e49ea957f9).

1. Create an account on [Firebase](https://firebase.google.com/)
2. Create a new project inside the Firebase console
3. Add an android app in your Firebase project
    - Follow the setup wizard
        - Add `google-services.json` to the `android/app/` folder
        - make changes to `android\build.gradle` and `android\app\build.gradle`
4. Add an IOS app in your Firebase project
    - Follow the setup wizard
        - Add `GoogleService-Info.plist` to the `ios/Runner/` folder
            - **Note:** If you do not run this from `XCode`, then be sure to include `GoogleService-Info.plist` to the `Runner` target inside `project.pbxproj`
                - This can be done by manually editing the `ios/Runner.xcodeproj/project.pbxproj` file as seen in [this commit](https://github.com/FlorisDevreese/Flutter-CI-CD/commit/27fe199ec4e5d5c6fc196cab2816d9e49ea957f9#diff-38c5d0dde42025b924ea1bc746fb2cca)
                - If you don't do this your app will crash on startup with this error
                    ```log
                    Could not locate configuration file: 'GoogleService-Info.plist'.
                    ```
                - **FYI:** I used [iTools](https://stackoverflow.com/a/26401867/3506115) for getting IOS logging on Windows 😉
        - You can skip the *Add Firebase SDK* step
            - I'm not sure, but it seems this is not needed with flutter
        - You can skip the *Add initialization code* step
            - I'm not sure, but it seems this is not needed with flutter
    - Edit `ios/Podfile` so you don't encounter [this build error](https://stackoverflow.com/q/59196199/3506115)
        ```log
        error: FirebaseCoreDiagnostics does not support provisioning profiles. FirebaseCoreDiagnostics does not support provisioning profiles, but provisioning profile Flutter_CI_CD__ad_hoc has been manually specified. Set the provisioning profile value to "Automatic" in the build settings editor. (in target 'FirebaseCoreDiagnostics' from project 'Pods')
        ```
        - Apply [this fix](https://stackoverflow.com/a/59206937/3506115)
        - **Note:** `ios/Podfile` will only be created once you run `flutter build ios`. If you don't have a `macOS` device you can get the created `Podfile` from your build pipeline as an artifact. You'll have to add this step to your pipeline, and the created `Podfile` will be available as an artifact of your build.
            ```yml
            - task: PublishBuildArtifacts@1
            displayName: Publish artifact
            inputs:
                PathtoPublish: '$(Build.SourcesDirectory)/ios/Podfile'
                ArtifactName: 'Podfile'
                publishLocation: 'Container'
            ````
5. Add the following dependencies to `pubspec.yml`
    ```yml
    firebase_core: ^0.4.4+3
    firebase_analytics: ^5.0.11
    ```

### Securing secrets

todo

## 4. Setting up CD for the Android package

First you'll need to do some manual steps before we can setup the automatic CD pipeline.

1. In your [Firebase](https://firebase.google.com/) project go to the distribution tab
    - Make sure you select the android app
2. The CI build created artifacts. One of those artifacts is `apk/app-release.apk`. Download that artifact to your device and upload it to App Distribution tab in Firebase
    - This will create a new release
    - Add testers to the release (e.g. your own email)
    - Next add some release notes
    - Next click Distribute
3. You should now receive an email with instruction how to install the application on your device
4. Follow those instructions on an Android device, and you should be able to run the application
    - **Note:** Appearently I forgot something because I can't install the application on my Android device. Give me some time to figure this out

Next we'll setup the CD pipeline. This so we can automatically push a new version of the app to our testers.

**Todo:** provide a CD pipeline

## 5. Setting up CD for the IOS package

Manual steps

1. In your [Firebase](https://firebase.google.com/) project go to the distribution tab
    - Make sure you select the ios app
2. The build created artifacts. One of those artifacts is `ios/Runner.ipa`. Download that artifact to your device and upload it to App Distribution tab in Firebase
    - This will create a new release
    - Add testers to the release (e.g. your own email)
    - Next add some release notes
    - Next click Distribute
3. You should now receive an email with instruction how to install the application on your device
4. Open that email on the ios device that you have registered in the [Apple developer console](https://developer.apple.com/account/resources/devices/list), and that you have added to the distribution profile
    - **Note:** the app will only work on the registered devices in your provisioning profile
    - **Note:** follow [these instructions](https://firebase.google.com/docs/app-distribution/ios/distribute-cli#step-3.-register-additional-tester-devices) if you want to add ios test devices 

Next we'll setup the CD pipeline. This so we can automatically push a new version of the app to our testers.

**Todo:** provide a CD pipeline