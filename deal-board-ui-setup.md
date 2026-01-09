# Deal Board UI Setup Guide

## Overview

Public "Deal Board" dashboard showing real estate deals in a card-based layout, color-coded by AI verdict. No authentication required.

---

## Option A: Airtable Interface Builder (Free)

### Step 1: Create Interface

1. Open your Airtable base: "Instant Deal Engine – Elvara"
2. Click **"Interfaces"** tab (left sidebar)
3. Click **"Create an interface"**
4. Select **"Start from scratch"**
5. Name: **"Deal Board"**
6. Click **"Create interface"**

### Step 2: Configure List Block

1. Click **"Add block"** or drag **"List"** block onto canvas
2. Select **"Deal Opportunities"** table as data source
3. In block settings panel (right side):

   **Display Options:**
   - **Layout:** Select **"Grid"** (card layout)
   - **Card size:** Medium or Large
   - **Records per page:** 20-50 (optional)

   **Field Selection:**
   - Click **"Add field"** or use field picker
   - Add fields in this order:
     1. **Property Title** (shown as card title)
     2. **Asking Price** (shown as subtitle/details)
     3. **AI Verdict** (shown as badge/tag)

   **Card Display:**
   - **Title:** Property Title
   - **Subtitle:** Asking Price (formatted as currency)
   - **Metadata/Badge:** AI Verdict

### Step 3: Apply Color Coding

1. In List block settings, find **"Conditional formatting"** or **"Color by field"**
2. Select **"AI Verdict"** field
3. Set color rules:
   - **🟢 BUY:** Green (#10B981 or similar)
   - **🔴 PASS:** Red (#EF4444 or similar)
   - **🟡 HUMAN REVIEW:** Yellow (#F59E0B or similar)

**Note:** If Airtable Interface doesn't support conditional card colors directly:
- Use **"Group by"** feature: Group by "AI Verdict" field
- Each group header will show verdict
- Cards within each group inherit visual grouping

### Step 4: Format Asking Price

1. In field display options for "Asking Price":
   - Ensure currency formatting is enabled
   - Format: USD ($)
   - Show as: Number with currency symbol

### Step 5: Configure Sharing (Public Access)

1. Click **"Share"** button (top right)
2. Select **"Share via link"**
3. Set permissions:
   - **Permission level:** "View only" or "Read only"
   - Enable **"Anyone with the link can view"**
4. Copy shareable link
5. Test link in incognito/private browser window

### Step 6: Optional Filters/Sorting

1. In List block settings:
   - **Default sort:** Property Title (A-Z) or Asking Price (Low to High)
   - **Filters:** Optional - filter by verdict if needed
   - **Search:** Enable search bar for property titles

### Step 7: Publish

1. Click **"Publish"** button (top right)
2. Interface is now live and accessible via share link
3. No login required for viewers

---

## Option B: Softr (Enhanced UI, Paid)

### Step 1: Create Softr App

1. Sign up/login at https://www.softr.io
2. Click **"Create new app"**
3. Select **"Start from scratch"** or **"Blank app"**
4. Name: **"Deal Board"**
5. Click **"Create app"**

### Step 2: Connect Airtable Base

1. In Softr dashboard, go to **"Data"** section
2. Click **"Add data source"**
3. Select **"Airtable"**
4. Authenticate with Airtable (OAuth)
5. Select base: **"Instant Deal Engine – Elvara"**
6. Select table: **"Deal Opportunities"**
7. Click **"Connect"**

### Step 3: Create List Page

1. In Softr app builder, click **"Add page"**
2. Select **"List"** page type
3. Name: **"Deal Board"** or **"All Deals"**
4. Select data source: **"Deal Opportunities"**

### Step 4: Configure Card Layout

1. In page settings, select **"Card view"** layout
2. Configure card fields:

   **Card Title:**
   - Field: **Property Title**
   - Size: Large/Bold

   **Card Subtitle/Details:**
   - Field: **Asking Price**
   - Format: Currency (USD)
   - Size: Medium

   **Card Badge/Tag:**
   - Field: **AI Verdict**
   - Display: Badge or Tag
   - Position: Top-right or bottom

### Step 5: Apply Color Coding

1. In card settings, find **"Conditional styling"** or **"Card colors"**
2. Set conditional rules based on **"AI Verdict"** field:

   **Rule 1:**
   - Condition: `AI Verdict = "🟢 BUY"`
   - Background color: Green (#10B981 or #22C55E)
   - Text color: White or dark green

   **Rule 2:**
   - Condition: `AI Verdict = "🔴 PASS"`
   - Background color: Red (#EF4444 or #DC2626)
   - Text color: White or dark red

   **Rule 3:**
   - Condition: `AI Verdict = "🟡 HUMAN REVIEW"`
   - Background color: Yellow (#F59E0B or #FBBF24)
   - Text color: Black or dark yellow

**Alternative Approach:**
- Use badge/tag styling with colors
- Set badge background color based on verdict value
- Cards remain white, badges show colors

### Step 6: Configure Public Access

1. Go to **"Settings"** → **"Access"** or **"Permissions"**
2. Set page/block permissions:
   - **Visibility:** Public (no authentication)
   - **Access level:** View only
3. Disable authentication requirement for this page

### Step 7: Publish App

1. Click **"Publish"** button (top right)
2. Get public URL
3. Test in incognito browser
4. No login required for viewers

---

## Field Display Configuration

### Property Title
- **Display:** Card title/heading
- **Font size:** Large (18-24px)
- **Weight:** Bold
- **Alignment:** Left

### Asking Price
- **Display:** Card subtitle or detail line
- **Format:** Currency with $ symbol (e.g., "$250,000")
- **Font size:** Medium (14-16px)
- **Position:** Below title

### AI Verdict
- **Display:** Badge, tag, or colored indicator
- **Format:** Show emoji + text (🟢 BUY)
- **Position:** Top-right corner or below price
- **Background:** Color-coded (green/red/yellow)
- **Text color:** White or dark (for contrast)

---

## Quick Setup Checklist

### Airtable Interface
- [ ] Created interface named "Deal Board"
- [ ] Added List block with Grid layout
- [ ] Selected Deal Opportunities table
- [ ] Added Property Title, Asking Price, AI Verdict fields
- [ ] Applied color coding by verdict (or grouped by verdict)
- [ ] Formatted Asking Price as currency
- [ ] Enabled public sharing (anyone with link)
- [ ] Tested public link in incognito browser
- [ ] Published interface

### Softr
- [ ] Created Softr app
- [ ] Connected Airtable base and table
- [ ] Created List page with Card view
- [ ] Configured card fields (Title, Price, Verdict)
- [ ] Applied conditional color coding by verdict
- [ ] Set page to public access (no auth)
- [ ] Published app
- [ ] Tested public URL

---

## Testing

1. **Verify Public Access:**
   - Open share link in incognito/private browser
   - Confirm no login required
   - Verify cards display correctly

2. **Verify Color Coding:**
   - Check that 🟢 BUY cards show green
   - Check that 🔴 PASS cards show red
   - Check that 🟡 HUMAN REVIEW cards show yellow

3. **Verify Data Display:**
   - Property Title shows correctly
   - Asking Price shows as currency ($XXX,XXX)
   - AI Verdict shows with emoji

4. **Test Updates:**
   - Add new record in Airtable
   - Verify it appears on Deal Board
   - Verify color coding applies correctly

---

## Troubleshooting

### Issue: Colors not showing
**Solution:** 
- Verify conditional formatting rules match exact verdict values (including emojis)
- Check field name matches exactly (case-sensitive)
- In Softr, ensure conditional styling is enabled

### Issue: Public link requires login
**Solution:**
- Airtable Interface: Check share settings, ensure "Anyone with link" is enabled
- Softr: Check page permissions, set to "Public" access level

### Issue: Currency not formatting
**Solution:**
- Verify Airtable field type is "Currency"
- In display options, ensure currency formatting is enabled
- Check number format matches USD ($)

### Issue: Cards not displaying
**Solution:**
- Verify table has records
- Check field names match exactly (case-sensitive)
- Verify data source connection is active
- Refresh browser/app

---

## URL Access

### Airtable Interface
- Share link format: `https://airtable.com/appXXXXXXXXXXXXXX/...`
- No authentication required if sharing enabled correctly
- Updates automatically as Airtable records change

### Softr
- App URL format: `https://your-app-name.softr.app`
- Or custom domain if configured
- Updates automatically as Airtable records change

---

## Maintenance

- **No code changes needed** - UI updates automatically from Airtable
- **Add/remove fields:** Update card display configuration in builder
- **Change colors:** Update conditional formatting rules
- **Modify layout:** Adjust card size, spacing, or layout type in builder
- **All changes:** Made through visual builder, no coding required
