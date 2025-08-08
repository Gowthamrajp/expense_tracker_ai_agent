# Bank Transactions Analysis Workflow

This n8n workflow automatically reads HDFC bank transaction emails from Gmail, extracts transaction data using regex patterns, and stores the data in Google Sheets for analysis.

## Features

- **Automated Email Processing**: Monitors Gmail for HDFC bank transaction emails
- **Smart Data Extraction**: Uses regex patterns to extract transaction details from HTML emails
- **Automatic Categorization**: Categorizes transactions as debit/credit
- **Google Sheets Integration**: Stores transaction data in a structured format
- **Real-time Updates**: Runs every hour to process new transactions
- **Comprehensive Data**: Extracts amount, merchant, card number, reference number, and more

## Workflow Overview

The workflow consists of 5 nodes:

1. **Schedule Trigger**: Runs every hour
2. **Get many messages**: Fetches HDFC transaction emails from Gmail
3. **Process Email Content**: Extracts HTML body and parses transaction details
4. **Extract Transaction Data**: Formats and validates extracted data
5. **Append row in sheet**: Stores transactions in Google Sheets

## Setup Instructions

### 1. Prerequisites

- n8n instance running (version 1.100.1+)
- Gmail account with HDFC bank transaction emails
- Google Sheets account
- Google Cloud Project with APIs enabled

### 2. Import the Workflow

1. Open your n8n instance at `http://localhost:5678`
2. Go to Workflows → Import from File
3. Upload the `expense monitor.json` file

### 3. Configure Authentication

#### Gmail OAuth2 Setup
1. In the "Get many messages" node, click on the authentication field
2. Create a new Gmail OAuth2 credential
3. Follow the OAuth2 setup process for Gmail API
4. Grant necessary permissions (read emails)

#### Google Sheets OAuth2 Setup
1. In the "Append row in sheet" node, configure OAuth2 authentication
2. Create a new Google Sheets OAuth2 credential
3. Grant permissions to read/write Google Sheets

### 4. Enable Google APIs

**IMPORTANT**: You must enable these APIs in your Google Cloud Console:

1. **Gmail API**: 
   - Visit: https://console.developers.google.com/apis/api/gmail.googleapis.com/overview
   - Click "Enable"

2. **Google Drive API**:
   - Visit: https://console.developers.google.com/apis/api/drive.googleapis.com/overview
   - Click "Enable"

### 5. Configure Google Sheets

Create a Google Sheet with the following structure:

#### Sheet: "Transactions"
```
| transaction_date | merchant | amount | category | transaction_type | description | email_id | extracted_at | card_number | reference_number |
|------------------|----------|--------|----------|------------------|-------------|----------|--------------|-------------|------------------|
```

### 6. Update Configuration

#### Google Sheet ID
Replace the `documentId` in the "Append row in sheet" node with your actual sheet ID:
- Current: `"1P5GrWjeG90lrrsEUeHTZnyLjv_hS2X4uTSoDDCvYar4"`
- Update to your sheet ID

#### Gmail Filter
The workflow is configured to fetch emails from HDFC bank:
- Current filter: `from:alerts@hdfcbank.net`
- Date filter: `receivedAfter: "2025-08-04T00:00:00"`

## Data Extraction

The workflow extracts the following data from HDFC transaction emails:

- **Amount**: Extracted from "Rs.XX.XX" format
- **Transaction Type**: Determined by "debited" or "credited" keywords
- **Merchant**: Extracted from text after "to" keyword
- **Card Number**: Extracted from "XX####" format
- **Date**: Extracted from "DD-MM-YY" format
- **Reference Number**: Extracted from "reference number is ####" format

## Recent Fixes

### ✅ Fixed Issues:

1. **HTML Body Extraction**: 
   - Fixed the "Process Email Content" node to correctly extract HTML from `email.json.html`
   - Previously was looking in wrong location (`payload`)

2. **Google Sheets Schema**: 
   - Added `card_number` and `reference_number` columns to schema
   - Updated column mapping to include all extracted fields

3. **Gmail Filter**: 
   - Updated to use proper Gmail API query format
   - Added date filter to avoid processing old emails

4. **Data Processing**: 
   - Improved regex patterns for better data extraction
   - Enhanced error handling for missing data

## Troubleshooting

### Common Issues

1. **"Gmail API has not been used in project"**:
   - Enable Gmail API in Google Cloud Console
   - Wait a few minutes for changes to propagate

2. **"Google Drive API has not been used in project"**:
   - Enable Google Drive API in Google Cloud Console
   - Wait a few minutes for changes to propagate

3. **"No columns found in Google Sheets"**:
   - Ensure your Google Sheet has the correct headers
   - Check that the sheet ID is correct
   - Verify OAuth2 permissions

4. **"No output in Extract Transaction Data"**:
   - Check if emails contain the expected HTML format
   - Verify regex patterns match your email structure
   - Test with a sample email

### Debugging Steps

1. **Test Gmail Node**: Run the "Get many messages" node to verify emails are fetched
2. **Check Email Content**: Verify the "Process Email Content" node outputs `htmlBody`
3. **Validate Extraction**: Ensure "Extract Transaction Data" produces transaction records
4. **Test Google Sheets**: Manually test the "Append row in sheet" node

## Security Considerations

- Store API keys securely in n8n credentials
- Use OAuth2 authentication for Gmail and Google Sheets
- Limit Google Sheets permissions to specific sheets
- Regularly review and clean up stored data

## Customization

### Email Filtering
Modify the Gmail filter in the "Get many messages" node:
```javascript
// Current filter
"filters": {
  "receivedAfter": "2025-08-04T00:00:00",
  "sender": "alerts@hdfcbank.net"
}

// Alternative filters
"additionalFields": {
  "q": "from:*hdfc* OR subject:transaction"
}
```

### Data Extraction
Modify the regex patterns in the "Process Email Content" node for different email formats:
```javascript
// Amount extraction
const amountMatch = cleanText.match(/Rs\.([\d,]+\.\d{2})/);

// Merchant extraction  
const merchantMatch = cleanText.match(/to\s+([^\s]+(?:\s+[^\s]+)*?)\s+on/);
```

### Schedule
Change the trigger frequency in the "Schedule Trigger" node:
- Every 15 minutes: `"field": "minutes"`
- Daily: `"field": "days"`
- Custom intervals available

## Support

For issues or questions:
1. Check n8n documentation at https://docs.n8n.io/
2. Review workflow execution logs in n8n
3. Test individual nodes to isolate issues
4. Verify all authentication credentials are properly configured

## License

This workflow is provided as-is for educational and personal use. 