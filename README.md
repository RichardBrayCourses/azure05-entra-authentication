# Azure 05 - Entra Authentication

Follow these steps in order.

Do not skip steps.

Run terminal commands from the repository root:

```bash
cd /Users/richardbray/src/azure05-entra-authentication
```

## Step 1: Install Tools

On macOS, install Homebrew if it is missing:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Install the required tools:

```bash
brew install node pnpm azure-cli gh
```

Check the tools:

```bash
node --version
pnpm --version
az version
gh --version
git --version
```

## Step 2: Sign In

Sign in to Azure:

```bash
az login
```

Check the selected Azure subscription:

```bash
az account show --output table
```

If the wrong subscription is selected:

```bash
az account set --subscription "<subscription-id-or-name>"
```

Sign in to GitHub:

```bash
gh auth login
```

Check GitHub:

```bash
gh auth status
gh repo view
```

## Step 3: Install The Project

Install packages:

```bash
pnpm install
```

Check TypeScript:

```bash
pnpm run type-check
```

Build the website:

```bash
pnpm run ui:build
```

## Step 4: Create The Entra App Registration

Open the Azure portal:

```text
https://portal.azure.com
```

Go to:

```text
Microsoft Entra ID
App registrations
New registration
```

Enter:

```text
Name: All Checks Out Azure05
Supported account types: Accounts in this organizational directory only
Redirect URI platform: Single-page application
Redirect URI: http://localhost:5173/auth/callback
```

Click:

```text
Register
```

Copy these values:

```text
Application (client) ID
Directory (tenant) ID
```

Go to:

```text
Authentication
Single-page application
Add URI
```

Add these redirect URIs:

```text
http://localhost:5173/auth/callback
https://testing.all-checks-out.com/auth/callback
https://staging.all-checks-out.com/auth/callback
https://www.all-checks-out.com/auth/callback
```

Click:

```text
Save
```

## Step 5: Configure Entra For Local Development

Create a local environment file:

```bash
touch apps/ui/.env.local
```

Open `apps/ui/.env.local`.

Add these lines:

```text
VITE_ENTRA_CLIENT_ID=<application-client-id>
VITE_ENTRA_AUTHORITY=https://login.microsoftonline.com/<directory-tenant-id>
```

If you have an API scope, add this line too:

```text
VITE_ENTRA_API_SCOPE=<api-scope>
```

Run the local website:

```bash
pnpm run ui:dev
```

Open:

```text
http://localhost:5173
```

Stop the local website when you are done:

```text
Control + C
```

## Step 6: Configure Entra For GitHub Actions

Set the GitHub repository variables:

```bash
gh variable set VITE_ENTRA_CLIENT_ID --body "<application-client-id>"
gh variable set VITE_ENTRA_AUTHORITY --body "https://login.microsoftonline.com/<directory-tenant-id>"
```

If you have an API scope:

```bash
gh variable set VITE_ENTRA_API_SCOPE --body "<api-scope>"
```

Check the variables:

```bash
gh variable list
```

## Step 7: Create Or Repair The Course Branches

Run:

```bash
pnpm run repo:init
```

If the GitHub remote is missing, run this instead:

```bash
pnpm run repo:init <github-url>
```

Check the branches:

```bash
git branch --list
```

You should see:

```text
main
testing
staging
production
```

## Step 8: Create The GitHub Actions Azure Credential

Use this repo prefix code:

```bash
REPO_PREFIX_CODE=azure05
```

Run:

```bash
APP_PREFIX="all-checks-out-$REPO_PREFIX_CODE-github-actions" pnpm run setup:github-azure
```

If these instructions are copied into a later repo, change the prefix code.

Examples:

```text
azure04
azure05
azure06
```

Check the GitHub secret:

```bash
gh secret list
```

You should see:

```text
AZURE_CREDENTIALS
```

## Step 9: Commit Everything

Check the files:

```bash
git status
```

Add the files:

```bash
git add .
```

Commit:

```bash
git commit -m "Prepare Azure05 phased delivery"
```

Push `main`:

```bash
git push -u origin main
```

## Step 10: Release To Testing

Run:

```bash
pnpm run release:testing
```

Wait for GitHub Actions:

```bash
pnpm run testing:wait-for-deploy
```

Get the testing DNS target:

```bash
pnpm run testing:get-storage-account
```

Copy the value it prints.

In Cloudflare, create or edit this DNS record:

```text
Type: CNAME
Name: testing
Target: <the value printed by pnpm run testing:get-storage-account>
Proxy status: DNS only
```

Wait until DNS is ready:

```bash
dig +short CNAME testing.all-checks-out.com
```

The command must print the same target value.

Connect the testing domain in Azure:

```bash
pnpm run testing:connect-domain
```

In Cloudflare, edit the same record:

```text
Proxy status: Proxied
```

Open:

```text
https://testing.all-checks-out.com
```

## Step 11: Release To Staging

Run:

```bash
pnpm run release:staging
```

Wait for GitHub Actions:

```bash
pnpm run staging:wait-for-deploy
```

Get the staging DNS target:

```bash
pnpm run staging:get-storage-account
```

Copy the value it prints.

In Cloudflare, create or edit this DNS record:

```text
Type: CNAME
Name: staging
Target: <the value printed by pnpm run staging:get-storage-account>
Proxy status: DNS only
```

Wait until DNS is ready:

```bash
dig +short CNAME staging.all-checks-out.com
```

The command must print the same target value.

Connect the staging domain in Azure:

```bash
pnpm run staging:connect-domain
```

In Cloudflare, edit the same record:

```text
Proxy status: Proxied
```

Open:

```text
https://staging.all-checks-out.com
```

## Step 12: Release To Production

Run:

```bash
pnpm run release:production
```

Wait for GitHub Actions:

```bash
pnpm run production:wait-for-deploy
```

Get the production DNS target:

```bash
pnpm run production:get-storage-account
```

Copy the value it prints.

In Cloudflare, create or edit this DNS record:

```text
Type: CNAME
Name: www
Target: <the value printed by pnpm run production:get-storage-account>
Proxy status: DNS only
```

Wait until DNS is ready:

```bash
dig +short CNAME www.all-checks-out.com
```

The command must print the same target value.

Connect the production domain in Azure:

```bash
pnpm run production:connect-domain
```

In Cloudflare, edit the same record:

```text
Proxy status: Proxied
```

Open:

```text
https://www.all-checks-out.com
```

## Normal Release Commands

Use these after the first setup is finished.

Testing:

```bash
pnpm run release:testing
pnpm run testing:wait-for-deploy
```

Staging:

```bash
pnpm run release:staging
pnpm run staging:wait-for-deploy
```

Production:

```bash
pnpm run release:production
pnpm run production:wait-for-deploy
```

## Useful Check Commands

Check Azure:

```bash
az account show --output table
```

Check GitHub:

```bash
gh auth status
gh repo view
```

Check branches:

```bash
git branch --list
```

Check GitHub Actions runs:

```bash
gh run list --workflow deploy.yml
```

Preview Azure changes:

```bash
pnpm run whatif:testing
pnpm run whatif:staging
pnpm run whatif:production
```

## Delete Azure Environments

Testing:

```bash
pnpm run destroy:testing
```

Staging:

```bash
pnpm run destroy:staging
```

Production:

```bash
pnpm run destroy:production
```

For production, type this when asked:

```text
DELETE-PRODUCTION
```
