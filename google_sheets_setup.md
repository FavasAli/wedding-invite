# Google Sheets Integration Setup Guide

## Step-by-Step Instructions

### 1. Create a Google Sheet
- Go to [Google Sheets](https://sheets.google.com)
- Create a new spreadsheet
- Name it "Wedding RSVP Responses"
- Add these column headers in the first row:
  - A1: **Timestamp**
  - B1: **Name**
  - C1: **Number of Guests**
  - D1: **Response** (Accepted/Declined)

### 2. Create Google Apps Script
- In your Google Sheet, go to **Extensions → Apps Script**
- Delete any existing code
- Paste this **IMPROVED code** (it updates existing entries instead of creating duplicates):

```javascript
function doPost(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    var data = JSON.parse(e.postData.contents);
    
    // Check if this person already submitted
    var dataRange = sheet.getDataRange();
    var values = dataRange.getValues();
    var nameColumn = 1; // Column B (Name)
    var updated = false;
    
    // Skip header row, search for existing name
    for (var i = 1; i < values.length; i++) {
      if (values[i][nameColumn] && values[i][nameColumn].toString().toLowerCase() === data.name.toLowerCase()) {
        // Update existing row
        sheet.getRange(i + 1, 1, 1, 4).setValues([[
          new Date(data.timestamp),
          data.name,
          data.guests,
          data.response
        ]]);
        updated = true;
        break;
      }
    }
    
    // If not found, add new row
    if (!updated) {
      sheet.appendRow([
        new Date(data.timestamp),
        data.name,
        data.guests,
        data.response
      ]);
    }
    
    return ContentService.createTextOutput(JSON.stringify({status: 'success', updated: updated}));
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({status: 'error', message: error.toString()}));
  }
}
```

### 3. Deploy as Web App
- Click the **Deploy** button (top right)
- Select **New deployment**
- Click the gear icon ⚙️ next to "Select type"
- Choose **Web app**
- Fill in the settings:
  - **Description**: Wedding RSVP Form Handler
  - **Execute as**: Me (your email)
  - **Who has access**: Anyone
- Click **Deploy**
- **Copy the Web App URL** - it will look like:
  `https://script.google.com/macros/s/AKfycby.../exec`

### 4. Update Your HTML File
- Open `wedding_invitation (1).html`
- Find this line (around line 540):
  ```javascript
  const GOOGLE_SHEET_URL = 'YOUR_GOOGLE_APPS_SCRIPT_URL_HERE';
  ```
- Replace `'YOUR_GOOGLE_APPS_SCRIPT_URL_HERE'` with your Web App URL:
  ```javascript
  const GOOGLE_SHEET_URL = 'https://script.google.com/macros/s/AKfycby.../exec';
  ```

### 5. Test It!
- Open your wedding invitation HTML file
- Fill out the RSVP form
- Click "Joyfully Accept" or "Decline"
- Check your Google Sheet - a new row should appear with the response!

---

## What's Changed in Your Invitation

✅ **Removed**: The public attendance count display  
✅ **Added**: All responses go to Google Sheets  
✅ **Smart Updates**: If someone submits again with the same name, their row gets updated (no duplicates!)  
✅ **User-Friendly Edit**: "Go Back & Edit" button on greeting page - no password needed  
  - Users can go back to the home page and resubmit with corrected information
  - The Google Sheet automatically updates their existing entry

---

## Troubleshooting

### Not receiving responses?
1. Make sure you deployed as "Anyone" can access
2. Check the Apps Script execution logs (View → Executions)
3. Verify the URL is correct in the HTML file

### User wants to change their response?
- They can click "Go Back & Edit" on the greeting page
- This reloads the home page where they can resubmit
- The Google Sheet will automatically update their existing row (no duplicates)

---

## Google Sheet Format

Your responses will look like this:

| Timestamp | Name | Number of Guests | Response |
|-----------|------|------------------|----------|
| 5/23/2026 14:30:25 | John Doe | 2 | Accepted |
| 5/23/2026 14:45:12 | Jane Smith | 1 | Accepted |
| 5/23/2026 15:10:03 | Bob Wilson | 0 | Declined |

**Smart Updates:** If "John Doe" goes back and changes his guest count from 2 to 3, his row will be **updated** (not duplicated):

| Timestamp | Name | Number of Guests | Response |
|-----------|------|------------------|----------|
| 5/23/2026 15:20:18 | John Doe | 3 | Accepted |
| 5/23/2026 14:45:12 | Jane Smith | 1 | Accepted |
| 5/23/2026 15:10:03 | Bob Wilson | 0 | Declined |

You can then:
- Filter by Response (Accepted/Declined)
- Sum the "Number of Guests" column to get total attendance
- Export to Excel or PDF
- Create charts and reports
- Track when people last updated their response
