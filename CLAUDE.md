# CLAUDE.md — AEM Asset Selector Chrome Extension

## Project Overview

A Chrome extension that enables content authors to browse AEM (Adobe Experience Manager) DAM assets and use them in any web application — Adobe Target, Salesforce, Braze, Marketo, or any tool with image/URL inputs.

## Architecture

```
┌─ Chrome Extension (thin shell) ────────────────────────┐
│  background.js   → Opens side panel, relays messages    │
│  bridge.js       → Content script on hosted page,       │
│                     catches CustomEvents, sends to BG    │
│  content.js      → Content script on ALL pages,         │
│                     shows floating widget, handles drops  │
│  content.css     → Drop zone + floating widget styles    │
│  manifest.json   → MV3 manifest, no side_panel path     │
└────────────────────────────────────────────────────────┘

┌─ Hosted Page (assetselector.vyingdigital.com) ─────────┐
│  index.html      → Single-page app:                     │
│                     - Setup form (first-time config)     │
│                     - Adobe Asset Selector MFE           │
│                     - Asset tray with Copy/Send buttons  │
│                     - Constructs DM delivery URLs        │
│  _headers        → Cloudflare Pages headers (X-Frame)   │
└────────────────────────────────────────────────────────┘
```

## Message Flow: "Send to Page" Button

```
1. User clicks "Send to Page" in hosted page tray
2. Hosted page dispatches: window.dispatchEvent(new CustomEvent('aem-send-to-page', {detail: asset}))
3. bridge.js (content script on assetselector.vyingdigital.com) catches the event
4. bridge.js calls: chrome.runtime.sendMessage({type: 'AEM_SEND_ASSET_TO_PAGE', asset})
5. background.js receives, queries active tab, forwards to content.js
6. content.js on active tab receives: {type: 'AEM_SHOW_FLOATING_WIDGET', asset}
7. content.js injects floating widget with draggable thumbnail onto the page
8. User drags from floating widget thumbnail to any drop target on the page
```

## Key Technical Decisions

### Why the side panel navigates to an external URL
Chrome extension pages have strict CSP that blocks external scripts (like Adobe's SDK from experience.adobe.com). The side panel uses `chrome.sidePanel.setOptions({path: 'https://assetselector.vyingdigital.com'})` to load the hosted page directly, bypassing extension CSP entirely.

### Why we can't do direct drag-and-drop from side panel
The Chrome side panel and the main page content are separate browsing contexts. HTML5 drag events don't cross between them. Solution: the floating widget is injected INTO the main page's DOM by the content script, making drag-and-drop work within the same context.

### Why BroadcastChannel doesn't work here
BroadcastChannel only works between same-origin contexts. The side panel (assetselector.vyingdigital.com) and the active tab (e.g., experience.adobe.com) are different origins. We use the extension's messaging system (bridge.js → background.js → content.js) instead.

### Dynamic Media Delivery URL Construction
With DM OpenAPI enabled, the Asset Selector returns `repo:repositoryId` (e.g., `delivery-p181502-e1907767.adobeaemcloud.com`) and `repo:assetId` (e.g., `urn:aaid:aem:xxxx-xxxx`). The delivery URL is constructed as:
```
https://{repo:repositoryId}/adobe/assets/{repo:assetId}/as/{repo:name}
```
For images with modifiers:
```
https://{repo:repositoryId}/adobe/assets/{repo:assetId}/as/{repo:name}?width=800&quality=80
```

## File Descriptions

### Extension Files

- **manifest.json**: MV3 manifest. No `side_panel.default_path` — background.js sets it dynamically to the external URL. Includes `host_permissions` for Adobe domains. Two content script entries: one for all URLs (content.js), one specifically for assetselector.vyingdigital.com (bridge.js).

- **background.js**: Sets side panel to external URL on install/startup. Relays messages from bridge.js to content.js on the active tab.

- **bridge.js**: Content script injected ONLY on assetselector.vyingdigital.com. Listens for `CustomEvent('aem-send-to-page')` dispatched by the hosted page's JavaScript. Forwards the asset data to background.js via `chrome.runtime.sendMessage`.

- **content.js**: Content script injected on ALL pages. Listens for `AEM_SHOW_FLOATING_WIDGET` messages from background. Creates a floating, repositionable widget with the asset thumbnail. Handles drag-start (sets multiple data formats), drop-zone highlighting, and drop handling (inserts URL/HTML into inputs, textareas, contenteditable, img elements).

- **content.css**: Styles for the floating widget and drop zone highlights. All rules use `!important` to override host page styles. Widget uses `z-index: 2147483647` to stay on top.

### Hosted Page Files

- **index.html**: Single HTML file deployed to Cloudflare Pages. Contains:
  - Adobe Asset Selector UMD script loaded from `experience.adobe.com`
  - Setup form for IMS credentials (stored in localStorage)
  - IMS auth registration (`PureJSSelectors.registerAssetsSelectorsAuthService`)
  - Asset Selector rendering (`PureJSSelectors.renderAssetSelectorWithAuthFlow`)
  - Asset tray with normalize function that extracts delivery URLs
  - "Send to Page" dispatches CustomEvent for bridge.js
  - "Copy URL" / "Copy IMG" / "Copy JSON" clipboard actions

- **_headers**: Cloudflare Pages header overrides. Removes X-Frame-Options and sets `frame-ancestors *` to allow iframing (needed if future versions return to iframe approach).

## Adobe Asset Selector MFE

The Asset Selector is Adobe's Micro-Frontend, loaded via UMD from:
```
https://experience.adobe.com/solutions/CQ-assets-selectors/static-assets/resources/assets-selectors.js
```

### Global: `PureJSSelectors`

Key methods:
- `registerAssetsSelectorsAuthService(imsProps)` — registers IMS OAuth. Must be called before render.
- `renderAssetSelectorWithAuthFlow(container, props, callback)` — renders the selector with built-in auth.

### IMS Props
```javascript
{
  imsClientId: 'your-client-id',
  imsScope: 'AdobeID,openid,additional_info.projectedProductContext,read_organizations',
  redirectUri: window.location.href,  // NOTE: redirectUri, not redirectUrl
  modalMode: true,
  onImsServiceInitialized: fn,
  onAccessTokenReceived: fn,
  onAccessTokenExpired: fn,
  onErrorReceived: fn
}
```

### Selector Props
```javascript
{
  imsOrg: 'ABC123@AdobeOrg',
  repositoryId: 'author-pXXXXX-eYYYYYY.adobeaemcloud.com',  // optional
  handleSelection: fn(assets),      // fired when user clicks "Select"
  handleAssetSelection: fn(assets), // fired as user toggles individual assets
  onClose: fn()
}
```

### Asset Object Shape (from handleSelection)
```javascript
{
  "dc:format": "image/jpeg",
  "repo:assetId": "urn:aaid:aem:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "repo:name": "image.jpg",
  "repo:path": "/content/dam/folder/image.jpg",
  "repo:repositoryId": "delivery-pXXXXX-eYYYYYY.adobeaemcloud.com",
  "repo:thumbnailUrl": "https://...",
  "tiff:imageWidth": 1920,
  "tiff:imageLength": 1080
}
```

## AEM Environment Details

- **Author**: `author-p181502-e1907767.adobeaemcloud.com`
- **Delivery**: `delivery-p181502-e1907767.adobeaemcloud.com`
- **Program ID**: p181502
- **Environment ID**: e1907767
- **IMS Org**: AA652270697124EB0A495E77@AdobeOrg
- **Hosted Page**: `assetselector.vyingdigital.com` (Cloudflare Pages)

## Version History

| Version | Approach | Outcome |
|---------|----------|---------|
| v1 | All-in-extension, chrome-extension:// origin | Worked but unstable extension ID |
| v2 | Iframe to hosted page with sandbox | Blank — sandbox blocked Adobe SDK |
| v3 | Direct SDK load in side panel | CSP blocked external scripts |
| v4 | Iframe without sandbox | "refused to connect" — X-Frame-Options |
| v4.1 | Iframe + _headers fix | Still refused — Chrome's own CSP for connect-src |
| v5 | window.location.href redirect | Stuck on spinner |
| v5.1 | setOptions with external URL | **WORKED** — Asset Selector loads in side panel |
| v6.0-6.2 | v5.1 + BroadcastChannel + floating widget | Widget appeared on wrong page (side panel) |
| v6.3 | v5.1 + bridge.js + background relay | Correct architecture — widget on active tab |
| v7 | v6.3 + DM delivery URLs + docs | Current version |

## Development Workflow

1. Edit `hosted/index.html` → push to GitHub → Cloudflare auto-deploys
2. Edit extension files → reload in `chrome://extensions/`
3. No build step needed — everything is vanilla JS/HTML/CSS
4. Test by opening any website + the extension side panel

## Important Notes

- Assets must be **Approved** in AEM for delivery URLs to work (DM OpenAPI requirement)
- The `redirectUri` in IMS props must match the domain registered in the Adobe support ticket
- `imsScope` must be `AdobeID,openid,additional_info.projectedProductContext,read_organizations` (comma-separated, no spaces)
- The IMS property is `redirectUri` (not `redirectUrl`) — this was a bug in early versions
- `apiKey` is NOT needed for `renderAssetSelectorWithAuthFlow`
