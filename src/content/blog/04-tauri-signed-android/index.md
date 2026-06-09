---
title: "How to sign your Android App with Tauri"
summary: "Sign your Android .APK with Tauri (step by step)"
date: "Jun 09 2026"
draft: false
tags:
  - Tauri
  - Android
  - APK
  - Signing
---

### So you're trying to install your Tauri app on your Android phone and Android tells you it's not trusted?

<div style="width: 100%;">
  <img src="https://i.pinimg.com/originals/57/9d/d4/579dd47e85363f4b5e100a51d73f806e.gif" alt="Android Tauri" style="width: 100%; height: auto; display: block;" />
</div>

The problem is probably that your APK isn't digitally signed. In this guide, I'll explain exactly how to do it.

## Why do you need to sign your APK?

Android requires all apps to be digitally signed for two reasons:

1. **Security**: The signature verifies that the APK hasn't been modified by third parties (no one injected malware)
2. **Identity**: Allows Google Play to verify that you are the real author

Without a signature, Android simply blocks the installation. It's a protection measure, not a whim.

## APK or AAB? Choose the right format

Before signing, decide which format you need:

### APK (Android Package Kit)

- The traditional format
- Can be installed directly on any phone
- Shared via WhatsApp, email, web, etc.
- **Use it for:** testing on your phone or distributing outside Google Play

### AAB (Android App Bundle)

- Google's modern format
- **Cannot be installed directly** — requires Google Play
- Automatically generates optimized APKs for each device
- **Use it for:** publishing on Google Play Store

## Step 1: Generate your keystore

> ⚠️ Before you continue, check the official Tauri documentation to make sure these instructions are still up to date:
> [v2.tauri.app/es/distribute/sign/android](https://v2.tauri.app/es/distribute/sign/android/)

A keystore is an encrypted file that contains your digital signature. **Keep it safe** — if you lose it, you won't be able to update your app on Play Store.

**macOS / Linux:**

```bash
keytool -genkey -v -keystore ~/upload-keystore.jks -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```

**Windows:**

```powershell
keytool -genkey -v -keystore $env:USERPROFILE\upload-keystore.jks -storetype JKS -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```

During the process, it will ask for:

- Keystore password
- Optional data (name, organization, etc.)
- Key password

**Tip:** Use the same password for keystore and key alias to simplify.

## Step 2: Configure signing properties

Create the file **src-tauri/gen/android/keystore.properties** with paths and passwords:

```properties
password=your_key_password
keyAlias=upload
storeFile=absolute/path/to/upload-keystore.jks
```

**Important:** Use the absolute path to the `.jks` file. For example on **macOS/Linux**:

```
storeFile=/home/your-name/projects/my-app/upload-keystore.jks
```

Or on **Windows**:

```
storeFile=C:\\Users\\<user name>\\upload-keystore.jks
```

## Step 3: Configure Gradle to use signing

Edit **src-tauri/gen/android/app/build.gradle.kts**.

Add the needed import at the beginning of the file:

```kotlin
import java.io.FileInputStream
```

Add the release signing config before the `buildTypes` block:

```kotlin
signingConfigs {
    create("release") {
        val keystorePropertiesFile = rootProject.file("keystore.properties")
        val keystoreProperties = java.util.Properties()
        if (keystorePropertiesFile.exists()) {
            keystoreProperties.load(FileInputStream(keystorePropertiesFile))
        }

        keyAlias = keystoreProperties["keyAlias"] as String
        keyPassword = keystoreProperties["password"] as String
        storeFile = file(keystoreProperties["storeFile"] as String)
        storePassword = keystoreProperties["password"] as String
    }
}
```

Use the new release signing config in the `buildTypes` block:

```kotlin
buildTypes {
    getByName("release") {
        signingConfig = signingConfigs.getByName("release")
    }
}
```

## Step 4: Build and sign

```bash
npm run tauri build
```

Tauri will generate these files:

| File                                   | What is it?                          |
| -------------------------------------- | ------------------------------------ |
| **app-universal-release.apk**          | ✅ Your signed APK, ready to install |
| **app-universal-release.aab**          | For uploading to Google Play         |
| **app-universal-release-unsigned.apk** | ❌ Unsigned, don't use it            |

## Install your APK on the phone

1. Transfer the **.apk** file to your phone
2. On Android, go to **Settings > Security** and enable **Unknown sources**
3. Open the **APK** file — it should install without problems!

## ⚠️ Critical warnings

- **Never upload the keystore to Git** — add it to `.gitignore`
- **Never share the passwords** anywhere
- **Make a backup** of the keystore in a safe place (USB, private cloud)
- If you lose the keystore, **you won't be able to update your app** on Play Store

## Resources

- Official Tauri documentation: [v2.tauri.app/distribute/sign/android](https://v2.tauri.app/distribute/sign/android/)

Take care of your keystore, friend.
