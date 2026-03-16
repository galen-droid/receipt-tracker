# Receipt Tracker

## What this project is

A web app that lets team members photograph or upload travel receipts on their phone, automatically extract key details via Google Cloud Vision OCR, and save the receipt image to Google Drive with the extracted data logged to a Google Spreadsheet.

## Tech stack

- Single `index.html` file — no build step, no bundler
- React 18 loaded via CDN (cdnjs.cloudflare.com)
- Babel standalone for JSX compilation in the browser
- All inline — no separate CSS or JS files
- Hosted on GitHub Pages

## Code structure (within index.html)

The file is organized into clearly separated sections:

1. **`<style>` block** — all CSS classes (layout, buttons, forms, modals, utilities)
2. **Constants** — config IDs, month names, currency codes, categories, headers
3. **Utility functions** — `extractId`, `getMonthLabel`, `fileToBase64`, `resizeImage`
4. **Receipt parsing** — decomposed into `extractMerchant`, `extractDate`, `extractTotal`, `extractCurrency`, `extractPayment`, `extractCategory`, `extractDescription`, composed by `parseReceipt`
5. **API functions** — `visionOCR`, `getUserInfo`, `findOrCreateFolder`, `ensureSheet`
6. **ErrorBoundary** — class component wrapping the app for graceful crash recovery
7. **UI components** — `GoogleIcon`, `FullScreenPreview`, `SettingsModal`, `SignInScreen`, `SessionHistory`, `CaptureScreen`, `ReviewForm`, `DoneScreen`
8. **App** — thin orchestrator owning state and delegating to components above

## Google APIs used (all via OAuth2 implicit flow)

- **Google Cloud Vision API** — OCR text extraction from receipt images
- **Google Drive API** — upload receipt files to organized folders
- **Google Sheets API** — log receipt data as rows
- **Google OAuth2 userinfo** — get logged-in user's name automatically

## Google Cloud project

- Project number: 186804789210
- OAuth Client ID: 186804789210-t6r2g6had74t7ven14nlt1pi7hl14onr.apps.googleusercontent.com
- Root Drive folder ID: 1IX1tvkckzHuyLx4OWK26RtpF_qAyyV9o
- Spreadsheet ID: 1gyG6r2Mp2y402G9IU6oJ7a9BLVQAzeVRm9OpA83apmg

## Drive folder structure

Receipts are organized as: `Root Folder > Month Year > Staff Name > receipt files`

Example:
```
📁 Root Folder
  📁 March 2026
    📁 Galen
      🧾 Chilis_2026-03-08_1741234567890.jpg
    📁 Sarah
      🧾 Uber_2026-03-10_1741234599999.pdf
  📁 April 2026
    ...
```

Monthly folders and staff subfolders are created automatically if they don't exist.

## Spreadsheet structure

Each monthly tab (e.g. "March 2026") is auto-created with these column headers:

`Staff | Date | Merchant | Category | Description | Total | Currency | Payment | Drive URL | Logged At`

## Authentication flow

- Uses OAuth2 **redirect** flow (not popup) for mobile compatibility
- Scopes: drive, spreadsheets, cloud-vision, userinfo.profile, userinfo.email
- Staff name is pulled automatically from Google account info
- Token is stored in React state (lost on page refresh — user re-authenticates)

## Receipt parsing logic

`parseReceipt()` composes individual extraction functions from OCR text:

- **Merchant**: first meaningful line, skipping noise words (receipt, cardholder, server, etc.)
- **Date**: supports YYYY-MM-DD, MM/DD/YYYY, DD-Mon-YYYY, M/D/YY formats
- **Total**: finds all "total" amounts, subtracts subtotals, picks the largest (to capture tip)
- **Currency**: checks for 3-letter codes first (CAD, USD, etc.), then symbols ($, €, £)
- **Payment**: detects Visa/Mastercard/Amex + last 4 digits, or Cash/Debit
- **Category**: auto-detects Food/Travel/Hotel/Entertainment from keywords
- **Description**: extracts menu item lines, filtering out totals/tax/payment noise

## Categories

Food, Travel, Hotel, Entertainment, Other

## Key design decisions

- Mobile-first — designed for iPhone use while traveling
- No server/backend — everything runs client-side
- No localStorage for sensitive data — auth tokens stay in memory only
- Staff name comes from Google account, not manual entry
- Image is resized to 1024px max before sending to Vision API (saves bandwidth)
- Full-resolution image kept separately for the "tap to enlarge" preview

## Known issues and areas for improvement

- OCR parser sometimes misses handwritten totals (tip amounts)
- Date parsing can fail on unusual receipt formats
- Token expires after ~1 hour with a 10-minute countdown warning; user must re-authenticate
- No offline support
- No batch upload (one receipt at a time)
- Description field sometimes pulls irrelevant lines from OCR

## Style preferences

- Clean, minimal UI — no unnecessary decoration
- Human-sounding copy — no AI-style writing markers
- Blue/indigo gradient header, white card body
- Mobile-optimized touch targets

## Git workflow

- `main` branch deploys automatically to GitHub Pages
- No build step required

## When making changes

- Use ES5-compatible syntax inside the Babel script block (var, function, .then() — avoid async/await). Exception: `class` is fine for React class components (ErrorBoundary)
- Test that JSX compiles correctly in the browser
- Don't introduce external dependencies beyond what's already loaded from cdnjs
- Preserve the existing Google API integration and OAuth flow
- Always maintain mobile-first responsive design
- Use CSS classes from the `<style>` block rather than inline styles for static layout
- Keep components focused — each component in its own clearly named function
- Parsing functions are individually named (`extractMerchant`, `extractDate`, etc.) — keep them separate rather than merging back into one function
