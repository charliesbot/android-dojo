# Google Cloud Setup for Play Store Deployment

This guide explains how to configure Google Cloud for automated Play Store deployments using GitHub Actions with Workload Identity Federation.

## Prerequisites

- Google Cloud account
- Google Play Console developer account
- GitHub repository with your Android app

## 1. Create a Google Cloud Project

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Click the project dropdown > **New Project**
3. Enter a project name (e.g., `android-play-store-automation`)
4. Click **Create**
5. Select the newly created project

## 2. Enable the Google Play Android Developer API

1. Go to [Android Publisher API](https://console.cloud.google.com/apis/library/androidpublisher.googleapis.com)
2. Click **Enable**

## 3. Create a Service Account

1. Go to **IAM & Admin** > **Service Accounts**
2. Click **Create Service Account**
3. Enter a name (e.g., `github-actions-deploy`)
4. Click **Create and Continue**
5. Skip granting permissions (click **Continue**)
6. Skip granting users access (click **Done**)
7. Note the service account email (e.g., `github-actions-deploy@your-project.iam.gserviceaccount.com`)

## 4. Set Up Workload Identity Federation

### Create the Workload Identity Pool

1. Go to **IAM & Admin** > **Workload Identity Federation**
2. Click **Create Pool**
3. Enter:
   - Name: `github-actions-pool`
   - Pool ID: `github-actions-pool`
4. Click **Continue**

### Add a Provider

1. Select **OpenID Connect (OIDC)**
2. Enter:
   - Provider name: `github-provider`
   - Provider ID: `github-provider`
   - Issuer URL: `https://token.actions.githubusercontent.com`
3. Click **Continue**

### Configure Attribute Mapping

Add these mappings:

| Google Attribute       | OIDC Attribute         |
| ---------------------- | ---------------------- |
| `google.subject`       | `assertion.sub`        |
| `attribute.actor`      | `assertion.actor`      |
| `attribute.repository` | `assertion.repository` |

### Configure Attribute Condition

To restrict access to only your GitHub organization/user:

```
assertion.repository.startsWith('YOUR_GITHUB_USERNAME/')
```

Replace `YOUR_GITHUB_USERNAME` with your GitHub username or organization name.

5. Click **Save**

### Note Your Pool Details

After creation, note the full workload identity provider resource name. It follows this format:

```
projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/github-actions-pool/providers/github-provider
```

You can find your project number in **Cloud Overview** > **Dashboard**.

## 5. Grant Repository Access to Service Account

For each repository that needs to deploy, you must add a Workload Identity User binding.

> **Important:** Wildcard bindings (`/*`) do not work with attribute-based principals. Each repository requires its own binding.

### Add a Repository Binding

1. Go to **IAM & Admin** > **Service Accounts**
2. Click on your service account (e.g., `github-actions-deploy`)
3. Go to the **Principals with access** tab
4. Click **Grant Access**
5. In **New principals**, enter:

   ```
   principalSet://iam.googleapis.com/projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/github-actions-pool/attribute.repository/GITHUB_USERNAME/REPO_NAME
   ```

   Replace:
   - `PROJECT_NUMBER` with your GCP project number
   - `GITHUB_USERNAME` with your GitHub username or organization
   - `REPO_NAME` with your repository name

6. In **Role**, select **Workload Identity User**
7. Click **Save**

### Example

For a repository `your-username/my-app` with project number `123456789`:

```
principalSet://iam.googleapis.com/projects/123456789/locations/global/workloadIdentityPools/github-actions-pool/attribute.repository/your-username/my-app
```

## 6. Add Service Account to Google Play Console

1. Go to [Google Play Console](https://play.google.com/console)
2. Select your developer account
3. Go to **Users and permissions**
4. Click **Invite new user**
5. Enter the service account email (e.g., `github-actions-deploy@your-project.iam.gserviceaccount.com`)
6. Set permissions:
   - Under **App permissions**, select your app
   - Grant **Release to production, exclude devices, and use Play App Signing**
7. Click **Invite user**
8. Click **Send invite**

> Note: It may take up to 24 hours for the service account permissions to propagate.

## 7. Configure GitHub Actions Workflow

Add these steps to your workflow:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write # Required for Workload Identity Federation

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Authenticate to Google Cloud
        id: auth
        uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: "projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/github-actions-pool/providers/github-provider"
          service_account: "github-actions-deploy@YOUR_PROJECT.iam.gserviceaccount.com"
          create_credentials_file: true
          export_environment_variables: true

      # ... build steps ...

      - name: Upload to Play Store
        uses: r0adkll/upload-google-play@v1
        with:
          serviceAccountJson: ${{ steps.auth.outputs.credentials_file_path }}
          packageName: com.example.app
          releaseFiles: app/build/outputs/bundle/release/app-release.aab
          track: alpha
```

Replace:

- `PROJECT_NUMBER` with your GCP project number
- `YOUR_PROJECT` with your GCP project ID

## Adding a New Repository

To add deployment support for a new repository:

1. Go to **IAM & Admin** > **Service Accounts** > your service account
2. Go to **Principals with access** tab
3. Click **Grant Access**
4. Add principal:
   ```
   principalSet://iam.googleapis.com/projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/github-actions-pool/attribute.repository/GITHUB_USERNAME/NEW_REPO_NAME
   ```
5. Assign role: **Workload Identity User**
6. Click **Save**

The new repository can now authenticate and deploy to the Play Store.

## Troubleshooting

### Error: "Permission 'iam.serviceAccounts.getAccessToken' denied"

This means the repository doesn't have a Workload Identity User binding. Add one following the steps in section 5.

### Error: "Malformed root json" for google-services.json

If using a base64-encoded secret for `google-services.json`, ensure you decode it properly:

```yaml
- name: Create google-services.json
  env:
    GOOGLE_SERVICES_JSON_BASE64: ${{ secrets.GOOGLE_SERVICES_JSON_BASE64 }}
  run: |
    printf '%s' "$GOOGLE_SERVICES_JSON_BASE64" | base64 --decode > app/google-services.json
```

Use `printf '%s'` instead of `echo` to avoid adding a trailing newline.

### Error: "base64: invalid input"

Re-encode your JSON file properly:

```bash
# macOS
base64 -i google-services.json | tr -d '\n'

# Linux
base64 -w 0 google-services.json
```

Copy the entire output (no line breaks) into your GitHub secret.
