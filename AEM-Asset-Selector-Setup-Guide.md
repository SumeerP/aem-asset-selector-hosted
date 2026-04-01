# AEM Asset Selector — Complete Setup Guide

## Table of Contents

1. Prerequisites
2. Part A: Deploy the hosted page to Cloudflare Pages
3. Part B: Install the Chrome extension
4. Part C: Configure AEM credentials
5. Part D: For other companies using this extension
6. Troubleshooting

---

## 1. Prerequisites

Before starting, make sure you have:

- **A Cloudflare account** with `vyingdigital.com` already managed (you have this)
- **A GitHub account** (you have this — `SumeerP`)
- **Google Chrome** browser (version 116+ for side panel support)
- **AEM as a Cloud Service** access (you have `author-p181502-e1907767.adobeaemcloud.com`)
- **Adobe Admin Console** access (to file the provisioning ticket)

---

## 2. Part A: Deploy the Hosted Page to Cloudflare Pages

This creates `assetselector.vyingdigital.com` — the permanent domain that all companies will use.

### Step 1: Create a GitHub repo for the hosted page

```bash
# On your local machine
mkdir aem-asset-selector-hosted
cd aem-asset-selector-hosted

# Copy the hosted/index.html from the zip into this folder
cp /path/to/aem-asset-selector-final/hosted/index.html .

# Initialize and push
git init
git add .
git commit -m "AEM Asset Selector hosted page"
git branch -M main
git remote add origin https://github.com/SumeerP/aem-asset-selector-hosted.git
git push -u origin main
```

### Step 2: Create a Cloudflare Pages project

1. Log into the **Cloudflare Dashboard** at `dash.cloudflare.com`
2. In the left sidebar, click **Workers & Pages**
3. Click **Create** (the blue button)
4. Select the **Pages** tab
5. Click **Connect to Git**
6. Select **GitHub** and authorize if prompted
7. Find and select the `aem-asset-selector-hosted` repository
8. Configure the build settings:
   - **Project name**: `aem-asset-selector` (this becomes `aem-asset-selector.pages.dev`)
   - **Production branch**: `main`
   - **Build command**: leave **empty** (it's a static HTML file, no build needed)
   - **Build output directory**: `/` (root — since index.html is at the root)
9. Click **Save and Deploy**
10. Wait 30-60 seconds for the first deployment to complete

At this point, your page is live at `https://aem-asset-selector.pages.dev`. You can verify by visiting this URL.

### Step 3: Add the custom subdomain `assetselector.vyingdigital.com`

Since `vyingdigital.com` is already managed in Cloudflare, this is straightforward:

1. In the Cloudflare dashboard, go to **Workers & Pages**
2. Click on your `aem-asset-selector` project
3. Click the **Custom domains** tab
4. Click **Set up a custom domain**
5. Enter: `assetselector.vyingdigital.com`
6. Click **Continue**
7. Cloudflare will show you a confirmation — it will **automatically create a CNAME DNS record** since `vyingdigital.com` is already on Cloudflare
8. Click **Activate domain**

The CNAME record Cloudflare creates looks like this:

| Type | Name | Target | Proxy |
|------|------|--------|-------|
| CNAME | assets | aem-asset-selector.pages.dev | Proxied |

9. Wait 1-2 minutes for SSL certificate provisioning
10. Visit `https://assetselector.vyingdigital.com` — you should see the AEM Asset Selector setup wizard

**That's it for deployment.** Any future updates just require pushing to the GitHub repo — Cloudflare auto-deploys on every push.

### Step 4: Verify the deployment

Open `https://assetselector.vyingdigital.com` in Chrome. You should see:

- A dark-themed page with "AEM Asset Selector" in the header
- A setup wizard with "Welcome to AEM Asset Selector" heading
- Two buttons: "I have credentials" and "I need to request credentials first"
- "Powered by Vying Digital" in the footer

If you see a 522 error, wait a few minutes for DNS propagation and try again.

---

## 3. Part B: Install the Chrome Extension

### For development (unpacked):

1. Unzip the `aem-asset-selector-final.zip` you downloaded
2. Open Chrome and go to `chrome://extensions/`
3. Toggle **Developer mode** ON (top-right corner)
4. Click **Load unpacked**
5. Navigate to and select the `extension/` folder (not the parent folder)
6. The extension appears in your extensions list with an ID like `abcdefghijklmnop...`
7. Click the **puzzle piece icon** in Chrome's toolbar (Extensions menu)
8. Find "AEM Asset Selector" and click the **pin icon** to keep it visible

### For production (Chrome Web Store):

1. Zip only the `extension/` folder contents:
   ```bash
   cd extension/
   zip -r ../aem-asset-selector-extension.zip .
   ```
2. Go to the [Chrome Web Store Developer Dashboard](https://chrome.google.com/webstore/devconsole)
3. Sign in with your Google account
4. Pay the one-time **$5 developer registration fee** (first time only)
5. Click **New Item** → upload the zip
6. Fill in the store listing:
   - Name: `AEM Asset Selector`
   - Description: (use the text from the README)
   - Category: Productivity
   - Language: English
7. Upload screenshots (take them after testing)
8. Submit for review — typically approved in **1-3 business days**
9. Once approved, anyone can install it from the Chrome Web Store

### Verify the extension works:

1. Click the extension icon in Chrome's toolbar
2. The side panel should open on the right side of the browser
3. You should see the hosted page loaded inside the panel — the same setup wizard from `assetselector.vyingdigital.com`

**Important:** If the iframe shows a blank page or error, check that:
- `assetselector.vyingdigital.com` is accessible in a regular browser tab
- The `src` URL in `extension/sidepanel.html` matches your actual deployed URL

---

## 4. Part C: Configure AEM Credentials

### Path 1: You already have credentials

If Adobe has already provisioned Asset Selector for your org:

1. Open the extension (click the icon)
2. Click **"I have credentials — Let's go"**
3. Enter:
   - **IMS Org ID**: Your Adobe org ID (e.g., `ABC123@AdobeOrg`)
     - Find it in: Adobe Admin Console → click your org name → profile
   - **IMS Client ID**: The client ID Adobe provided
   - **API Key**: The API key Adobe provided (often same as Client ID)
   - **Repository ID** (optional): `author-p181502-e1907767.adobeaemcloud.com`
   - **IMS Scope**: Leave the default unless Adobe specified otherwise
4. Click **Save & Connect**
5. The wizard moves to "You're all set!" — click **Launch Asset Selector**
6. An Adobe IMS login popup appears — log in with your Adobe ID
7. After authentication, the Asset Selector loads with your DAM

### Path 2: You need to request credentials first

1. Open the extension (click the icon)
2. Click **"I need to request credentials first"**
3. Fill in:
   - **Company Name**: Vying Digital
   - **AEM Author URL**: `https://author-p181502-e1907767.adobeaemcloud.com`
   - **Program ID**: `p181502`
   - **Environment ID**: `e1907767`
   - **Your Name**: Your name
4. Click **Generate Ticket**
5. The wizard generates a pre-formatted support ticket with `assetselector.vyingdigital.com` as the redirect URL
6. Click **Copy to clipboard**
7. Go to **Adobe Admin Console** → **Support** → **Create Ticket**
   - Paste the ticket text
   - Set priority to **P2**
8. Adobe responds with your `imsClientId`, `imsScope`, and `apiKey` (typically within a few business days)
9. Come back to the extension and enter the credentials in Path 1

### Important: What to tell Adobe

The critical line in the support ticket is the **redirect URL domain**. Adobe needs to allowlist:

```
https://assetselector.vyingdigital.com
```

This is non-negotiable — without this in the allowlist, the IMS OAuth popup will fail with an "invalid redirect" error.

---

## 5. Part D: For Other Companies

When another company wants to use this extension, here's their journey:

### What they do:

1. **Install** the extension from Chrome Web Store (or load unpacked for testing)
2. **Open** the extension → setup wizard appears
3. **Generate** the Adobe support ticket using the built-in wizard
   - They fill in their own company name, AEM URL, program ID, etc.
   - The generated ticket automatically includes `assetselector.vyingdigital.com` as the redirect domain
4. **File** the ticket through their Adobe Admin Console
5. **Wait** for Adobe to provision and respond (a few business days)
6. **Enter** their credentials in the setup wizard
7. **Start** browsing and dragging assets

### What you (Vying) don't need to do:

- No backend changes per customer
- No configuration on Cloudflare
- No Adobe coordination on their behalf
- No credential management

Each company self-serves entirely. The only shared infrastructure is the hosted page at `assetselector.vyingdigital.com`.

### Prerequisite for each company:

- AEM as a Cloud Service license (they must have this already)
- Adobe Admin Console access (to file the support ticket)
- Chrome browser

---

## 6. Troubleshooting

### Cloudflare deployment issues

| Problem | Solution |
|---------|----------|
| 522 error on assetselector.vyingdigital.com | Wait 2-5 minutes for DNS propagation. Check that the custom domain shows "Active" in the Cloudflare Pages dashboard. |
| Page shows "not found" | Ensure `index.html` is at the root of the repo, not in a subfolder. Check Build output directory is set to `/`. |
| SSL certificate error | Cloudflare auto-provisions SSL. Wait up to 15 minutes. If it persists, check for conflicting CAA records in your DNS. |
| Changes not reflecting | Cloudflare auto-deploys on git push. Check the deployment log in Workers & Pages → your project → Deployments. |

### Chrome extension issues

| Problem | Solution |
|---------|----------|
| Extension icon doesn't appear | Go to chrome://extensions/ and ensure the extension is enabled. Click the puzzle icon and pin it. |
| Side panel is blank | Check that assetselector.vyingdigital.com loads in a regular tab first. Verify the iframe src in sidepanel.html. |
| Side panel shows the page but tiny | This is normal for the first load. The side panel width is controlled by Chrome — users can resize by dragging the panel edge. |
| "Service worker registration failed" | Make sure you loaded the `extension/` folder, not the parent folder. |

### AEM / IMS authentication issues

| Problem | Solution |
|---------|----------|
| IMS popup blocked | Chrome blocks popups by default. Click the blocked popup icon in the address bar and allow popups for assetselector.vyingdigital.com. |
| "Invalid redirect URI" | The hosted domain is not in Adobe's IMS allowlist. Confirm the support ticket was processed and `assetselector.vyingdigital.com` was added. |
| "Invalid client ID" | Double-check the imsClientId you entered. It's case-sensitive. |
| Asset Selector loads but shows no assets | Check the Repository ID field — if set wrong, it points to a repo you don't have access to. Try leaving it blank to see all available repos. |
| Token expired error | Click the refresh button in the header. If it persists, clear the config (Settings → re-enter credentials). |

### Drag and drop issues

| Problem | Solution |
|---------|----------|
| Drop zones don't highlight on the target page | Refresh the target page after installing the extension. The content script needs to be injected. |
| Drag from tray does nothing | Cross-origin iframe drag has limitations. The fallback is: select an asset, then manually paste the URL. |
| URL pastes but image doesn't render | The AEM delivery URL needs CORS headers. Check your AEM Dispatcher/CDN configuration. |
