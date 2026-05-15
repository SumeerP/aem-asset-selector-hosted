# Project Context: AEM Asset Selector Chrome Extension

## What This Project Does

This Chrome extension allows marketing and content teams to browse their company's AEM (Adobe Experience Manager) digital asset library directly in Chrome's side panel, then use those assets in any web application they're working in — Adobe Target (for A/B testing), Salesforce (for email campaigns), Braze (for push notifications), or any tool that accepts image URLs.

## The Problem It Solves

Without this extension, content authors must: open AEM → find the asset → download it → switch to Target/Salesforce/Braze → upload it again. This wastes time and creates copies that get out of sync with the DAM.

With this extension: open the side panel → find the asset → click "Send to Page" or "Copy URL" → the asset's delivery URL is instantly available. No downloading, no re-uploading, no copies.

## Who Uses It

- **Content authors**: people creating marketing campaigns in Adobe Target
- **Email marketers**: people building emails in Salesforce or Braze
- **Brand managers**: people who need approved, on-brand assets used consistently
- **Any AEM customer**: this is designed as a multi-tenant product — each company enters their own Adobe credentials

## How It's Deployed

Two components:
1. **Hosted page** at `assetselector.vyingdigital.com` — a single HTML file on Cloudflare Pages that loads Adobe's Asset Selector MFE
2. **Chrome extension** — installed from a zip or Chrome Web Store, opens the hosted page in Chrome's side panel

## Key Dependencies

- **Adobe Asset Selector MFE** — Adobe's official micro-frontend for browsing DAM assets
- **Adobe IMS** — Adobe's identity management for authentication
- **Dynamic Media with OpenAPI** — Adobe's CDN delivery system for approved assets (provides public delivery URLs)
- **Chrome Side Panel API** — Chrome's built-in side panel (requires Chrome 116+)
- **Cloudflare Pages** — hosting for the single-page app

## Current Status

Working: Asset Selector loads in side panel, IMS auth works, asset browsing works, copy URL/IMG/JSON works.
In progress: Floating widget for drag-and-drop from side panel to active page, Dynamic Media delivery URL construction.
