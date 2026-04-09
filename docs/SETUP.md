# Application Frontend - Setup Guide

Step-by-step guide to deploying your first app to SAP Application Frontend.

## Prerequisites

- SAP BTP subaccount (trial or enterprise)
- Application Frontend service subscribed in the subaccount
- Cloud Foundry CLI (`cf`) installed
- Node.js 18+ installed

## Step 1: Create a Service Instance

Log in to Cloud Foundry and create an Application Frontend service instance:

```bash
cf login -a https://api.cf.eu10.hana.ondemand.com --sso
cf create-service app-front developer my-appfront-instance
```

> **Note:** The service plan may be `developer` or `standard` depending on your account type. Check with `cf marketplace -e app-front`.

## Step 2: Create a Service Key

```bash
cf create-service-key my-appfront-instance my-appfront-key
```

View the key and save the JSON output:

```bash
cf service-key my-appfront-instance my-appfront-key
```

Save the JSON output (everything between the curly braces) to a file, e.g. `service-key.json`. **Do not commit this file to git.**

## Step 3: Install the CLI

```bash
npm install -g @sap/appfront-cli
```

Verify installation:

```bash
afctl --version
```

## Step 4: Deploy Manually (first time)

```bash
# Login to Application Frontend
afctl login --service-key service-key.json -a https://api.eu10.dt.appfront.cloud.sap

# Deploy the webapp folder
afctl push webapp --activate

# Check your deployed apps
afctl list
```

The CLI will output a URL where your app is running.

## Step 5: Set Up GitHub Actions (automated deployment)

### 5a. Add the service key as a secret

1. Go to your GitHub repo > **Settings** > **Secrets and variables** > **Actions**
2. Click **New repository secret**
3. Name: `APPFRONT_SERVICE_KEY`
4. Value: paste the full JSON from `cf service-key` output

### 5b. Add the API URL as a variable

1. Same page, switch to the **Variables** tab
2. Click **New repository variable**
3. Name: `APPFRONT_API_URL`
4. Value: `https://api.eu10.dt.appfront.cloud.sap` (adjust for your region)

### 5c. Push to main

```bash
git add .
git commit -m "initial deploy"
git push origin main
```

The pipeline will automatically build and deploy your app. Check the **Actions** tab for progress.

## Step 6: Framework-Specific Setup

### React (Vite)

Add these GitHub **variables**:

| Variable | Value |
|----------|-------|
| `BUILD_COMMAND` | `npm run build` |
| `APP_BUILD_PATH` | `dist` |

Make sure `webapp/manifest.json` and `webapp/xs-app.json` are copied to `dist/` in your build, or let the pipeline auto-generate them.

### SAPUI5 / Fiori Elements

Add these GitHub **variables**:

| Variable | Value |
|----------|-------|
| `BUILD_COMMAND` | `npm run build` |
| `APP_BUILD_PATH` | `dist` |

### Next.js (Static Export)

Configure `output: 'export'` in `next.config.js`, then:

| Variable | Value |
|----------|-------|
| `BUILD_COMMAND` | `npm run build` |
| `APP_BUILD_PATH` | `out` |

## Troubleshooting

### "Service plan not found"

Run `cf marketplace -e app-front` to see available plans. Use the correct plan name in `cf create-service`.

### "Login failed"

Verify your service key JSON is valid and the API URL matches your region.

### "No manifest.json found"

The pipeline auto-generates one, but you can create it manually:

```json
{
  "sap.app": {
    "id": "my-app-name",
    "applicationVersion": {
      "version": "v1"
    }
  }
}
```

### Build fails

Make sure `package.json` and `package-lock.json` are committed. Run `npm ci && npm run build` locally first to verify.
