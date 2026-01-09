# Automated Google Play Store Deployment - Setup Guide

This guide explains how to set up automated weekly deployments to Google Play Store for Android apps using GitHub Actions with Workload Identity Federation.

## Overview

This setup uses:

- **GitHub Actions** for CI/CD
- **Workload Identity Federation** for secure, keyless authentication to Google Cloud
- **Google Play Publishing API** for automated uploads

Once configured, your app will automatically deploy to the Play Store on a schedule (default: every Monday at 10:00 AM UTC).

---

## Prerequisites

Before starting, make sure you have:

- [ ] Google Cloud project set up (see `google-cloud-setup.md`)
- [ ] Service account with Play Console permissions (see `google-play-console-setup.md`)
- [ ] App already published on Google Play Store (initial version uploaded manually)
- [ ] App has a release keystore (.jks file)
- [ ] GitHub repository created for the app
- [ ] Firebase project set up (optional, for Crashlytics)

---

## Setup Steps

### Step 1: Configure Version Properties in Gradle

Add version property support to your build files so the workflow can inject version codes dynamically.

**In `app/build.gradle.kts` (before `android {}` block):**

```kotlin
// Support for runtime version override from CI
val defaultVersionCode = java.text.SimpleDateFormat("yyMMdd").format(java.util.Date())
val versionCodeProperty: String = (project.findProperty("versionCode") as String?) ?: defaultVersionCode
val versionNameProperty: String = (project.findProperty("versionName") as String?) ?: "1.$defaultVersionCode-dev"
```

**Update `defaultConfig`:**

```kotlin
defaultConfig {
    applicationId = "com.example.yourapp"
    versionCode = versionCodeProperty.toInt()
    versionName = "$versionNameProperty-mobile"  // Add platform suffix
}
```

**If you have a Wear OS module** (e.g., `wearos/build.gradle.kts`), repeat the same but with:

```kotlin
defaultConfig {
    applicationId = "com.example.yourapp"  // Same as phone app
    versionCode = versionCodeProperty.toInt() + 1  // +1 to avoid conflicts
    versionName = "$versionNameProperty-wear"      // -wear suffix
}
```

---

### Step 2: Create Release Notes

Create release notes files that will be displayed in Play Store.

**Directory structure:**

```
.github/
  whatsnew/
    en-US.txt
    es-ES.txt
```

**Example `en-US.txt`:**

```
Bug fixes and performance improvements.

Thank you for using [Your App Name]!
```

**Example `es-ES.txt`:**

```
Corrección de errores y mejoras de rendimiento.

¡Gracias por usar [Your App Name]!
```

Add more locales as needed (`fr-FR.txt`, `de-DE.txt`, etc.)

---

### Step 3: Copy GitHub Actions Workflow

Copy the workflow template from `workflows/deploy-play-store.yml` to your project:

```bash
mkdir -p .github/workflows
cp path/to/android-dojo/github-actions/workflows/deploy-play-store.yml .github/workflows/
```

**Customize the workflow:**

1. Replace `YOUR_GCP_PROJECT_NUMBER` with your Google Cloud project number
2. Replace `YOUR_GCP_PROJECT_ID` with your Google Cloud project ID
3. Replace `YOUR_PACKAGE_NAME` with your app's package name
4. Update the build task if using different flavors
5. Update the AAB path to match your build output

**Important notes:**

- For **phone-only apps**, use the default template
- For **Wear OS apps**, add a second build and upload step with `wear:` track prefix
- For **other form factors** (TV, Automotive, XR), use the appropriate track prefix (see Appendix A)

---

### Step 4: Grant Workload Identity Pool Access

Allow your GitHub repository to impersonate the service account.

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Select your project
3. Navigate to **IAM & Admin** → **Workload Identity Federation**
4. Click on your pool (e.g., `github-actions-pool`)
5. Click **Grant Access** button
6. Select **"Grant access using service account impersonation"**
7. Fill in:
   - **Service account:** Your service account email
   - **Select principals:** Choose `repository` from dropdown
   - **Repository:** Enter `your-github-username/your-repo-name`
8. Click **Save**

---

### Step 5: Grant Play Console Permissions

Give the service account permission to release your specific app.

1. Go to https://play.google.com/console
2. Click **"Users and permissions"** (left sidebar)
3. Find your service account email
4. Click on it to edit
5. Under **"App permissions"**:
   - Click **"Add app"**
   - Select your app
   - Check **"Releases"**
6. Click **"Save"**

---

### Step 6: Prepare Secrets

Encode your keystore and Firebase config for GitHub Actions.

#### Keystore

Find your keystore file and encode it:

```bash
base64 -i path/to/your-keystore.jks | pbcopy  # macOS
base64 -w 0 path/to/your-keystore.jks         # Linux
```

You'll also need from your `keystore.properties`:

- Key alias (e.g., `key0`, `upload`)
- Keystore password
- Key password

#### google-services.json (Firebase - Optional)

Download from Firebase Console and encode:

```bash
base64 -i app/google-services.json | pbcopy  # macOS
base64 -w 0 app/google-services.json         # Linux
```

---

### Step 7: Add GitHub Secrets

1. Go to your GitHub repo → **Settings** → **Secrets and variables** → **Actions**
2. Click **"New repository secret"** for each:

| Secret Name                   | Value                   | Source              |
| ----------------------------- | ----------------------- | ------------------- |
| `KEYSTORE_FILE_BASE64`        | Base64 encoded keystore | Step 6              |
| `KEYSTORE_KEY_ALIAS`          | Key alias               | keystore.properties |
| `KEYSTORE_STORE_PASSWORD`     | Keystore password       | keystore.properties |
| `KEYSTORE_KEY_PASSWORD`       | Key password            | keystore.properties |
| `GOOGLE_SERVICES_JSON_BASE64` | Base64 encoded JSON     | Step 6 (optional)   |

---

### Step 8: Declare Foreground Service Permissions (If Applicable)

**Skip this step if your app doesn't use foreground services.**

Check your `AndroidManifest.xml` for:

- `<uses-permission android:name="android.permission.FOREGROUND_SERVICE*" />`
- `<service android:foregroundServiceType="..." />`

If found, declare in Play Console:

1. Go to Play Console → Your App → **Policy** → **App content**
2. Find **"Foreground service permissions"**
3. Click **Manage**
4. Select the types your app uses (HEALTH, DATA_SYNC, etc.)
5. Provide use case descriptions
6. Save

**Important:** Declarations must match your manifest or uploads will fail.

---

### Step 9: Test the Deployment

1. Go to GitHub repo → **Actions** tab
2. Click **"Deploy to Google Play Store"** workflow
3. Click **"Run workflow"**
4. Leave version suffix empty (auto-adds hour suffix)
5. Click **"Run workflow"** button
6. Monitor the build (~15-20 minutes)

**Verify in Play Console:**

1. Go to https://play.google.com/console
2. Select your app
3. Check **Release** → **Production** (and **Wear OS** tab if applicable)
4. You should see a new release

**If you see "Ready for review":** Click "Send for review" manually. After several successful reviews, Google usually restores auto-publishing.

---

## Done!

Your app will now automatically deploy **every Monday at 10:00 AM UTC**.

**For hotfixes:**

- Go to Actions → "Deploy to Google Play Store" → "Run workflow"
- Enter custom version suffix (`1`, `2`, etc.) or leave empty for auto hour suffix

---

## Appendix A: Form Factor Tracks

Since March 2023, Google requires non-phone apps to use dedicated tracks with prefixes.

**Track Format:** `[prefix]:trackName`

**Supported Prefixes:**

- **(none)** - Phone/tablet apps → `track: production`
- `wear:` - Wear OS → `track: wear:production`
- `automotive:` - Android Automotive OS → `track: automotive:production`
- `tv:` - Android TV → `track: tv:production`
- `xr:` - Android XR → `track: xr:production`

**Key Points:**

- Phone apps use NO prefix
- All other form factors REQUIRE the prefix
- Apps with same package name go to separate tracks
- The API routes binaries based on track prefix

---

## Appendix B: Version Strategy

### How Versions Work

**Phone app:**

- versionCode: `251208` (YYMMDD)
- versionName: `1.251208-mobile`

**Wear OS app:**

- versionCode: `251209` (YYMMDD + 1)
- versionName: `1.251208-wear`

### Manual Runs

- Auto-appends hour suffix: `25120814` (Dec 8, 2025 at 14:00)
- Version names: `1.25120814-mobile`, `1.25120814-wear`

### Scheduled Runs

- Clean date format: `251208`
- Version names: `1.251208-mobile`, `1.251208-wear`

### Custom Suffixes

- Enter `1`, `2`, `3` in workflow input
- Creates: `2512081`, `2512082`, etc.

---

## Troubleshooting

### Permission Errors

**"Permission 'iam.serviceAccounts.getAccessToken' denied"**

- Complete Step 4 (Workload Identity Pool access)
- Verify correct repository name in grant

**"Service account not authorized"**

- Complete Step 5 (Play Console permissions)
- Verify "Releases" permission is checked

### Upload Errors

**"You must declare foreground service permissions"**

- Complete Step 8
- Ensure declarations match your AndroidManifest.xml

**"Version code must be greater than X"**

- Use a version suffix for same-day deploys
- Or wait until tomorrow for higher date-based version

**"APKs must have unique version codes"**

- Ensure Wear OS uses `versionCodeProperty.toInt() + 1`

### Build Errors

**"File google-services.json is missing"**

- Add `GOOGLE_SERVICES_JSON_BASE64` secret (Step 7)
- Verify Base64 encoding is correct

**"versionCode property is required"**

- This is expected for local builds
- Workflow provides these automatically

---

## Resources

- [r0adkll/upload-google-play](https://github.com/r0adkll/upload-google-play)
- [Form Factor Tracks](https://support.google.com/googleplay/android-developer/answer/13295490)
- [Google Play Publishing API](https://developers.google.com/android-publisher)
- [Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation)
- [Foreground Service Types](https://developer.android.com/about/versions/14/changes/fgs-types-required)
