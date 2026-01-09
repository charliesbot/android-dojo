# Google Play Console Setup for Service Account

This guide explains how to grant your GCP service account permission to upload apps to Google Play Console.

## Prerequisites

- Google Play Console developer account
- Service account already created in Google Cloud (see `google-cloud-setup.md`)

## Add Service Account to Google Play Console

1. Go to [Google Play Console](https://play.google.com/console)
2. Select your developer account
3. Go to **Users and permissions** (left sidebar)
4. Click **Invite new user**
5. Enter the service account email:
   ```
   github-actions-deploy@your-project.iam.gserviceaccount.com
   ```

## Configure Permissions

### Account Permissions

No account-level permissions are required. Skip this section.

### App Permissions

1. Click **Add app**
2. Select your app (e.g., `com.example.yourapp`)
3. Grant the appropriate permission:

| Permission                         | Use Case                                |
| ---------------------------------- | --------------------------------------- |
| **Release apps to testing tracks** | Deploy to alpha, beta, internal testing |
| **Release to production**          | Deploy to production                    |

For CI/CD to testing tracks (alpha/beta), select **Release apps to testing tracks**.

4. Click **Apply**
5. Click **Invite user**
6. Click **Send invite**

## Adding Multiple Apps

If you have multiple apps (e.g., different flavors with different package names), repeat the app permissions step for each:

1. Go to **Users and permissions**
2. Click on the service account
3. Go to **App permissions** tab
4. Click **Add app**
5. Select the additional app
6. Grant permissions
7. Click **Save**

### Example: Multiple Flavors

If your app has flavors like:

- `com.example.yourapp` (prod)
- `com.example.yourapp.nightly` (nightly)

Each is treated as a separate app in Play Console and needs permissions granted individually.

## Permission Propagation

After granting permissions, it may take up to **24 hours** for changes to fully propagate. If you get permission errors immediately after setup, wait and try again.

## Troubleshooting

### Error: "The caller does not have permission"

This means the service account is authenticated but lacks Play Console permissions.

1. Verify the service account email is correct in Play Console
2. Check that app permissions are granted for the specific package name
3. Wait for permission propagation (up to 24 hours)

### App Not Listed in App Permissions

The app must exist in Play Console before you can grant permissions. Create the app first:

1. Go to **All apps** > **Create app**
2. Fill in app details
3. Complete the initial setup
4. Then return to grant service account permissions
