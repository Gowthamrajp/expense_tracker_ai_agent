# n8n Bank Transaction Workflow

An automated workflow built with n8n to extract and analyze bank transaction data from HDFC Bank emails.

## Features

- **Automated Email Processing**: Retrieves HDFC Bank transaction emails automatically
- **Smart Data Extraction**: Uses regex patterns to extract transaction details from HTML email bodies
- **Failed Transaction Tracking**: Captures and logs transactions that couldn't be processed for debugging
- **Google Sheets Integration**: Stores both successful and failed transactions in Google Sheets
- **Scheduled Execution**: Runs automatically every hour to process new emails

## Workflow Components

### Nodes
1. **Schedule Trigger**: Runs the workflow every hour
2. **Get many messages (Gmail)**: Retrieves HDFC Bank emails with filters
3. **Process Email Content (Code)**: Extracts transaction details from HTML email bodies
4. **Extract Transaction Data (Code)**: Processes and formats transaction data
5. **Append row in sheet (Google Sheets)**: Stores successful transactions
6. **Append Failed Transactions (Google Sheets)**: Stores failed transactions for debugging

### Data Extraction
The workflow extracts the following data points:
- Transaction date
- Merchant name
- Amount
- Transaction type (debit/credit)
- Card number (masked)
- Reference number
- Category (Purchase/Refund)
- Description

## Setup Instructions

### Prerequisites
- n8n installed and running
- Gmail account with HDFC Bank emails
- Google Sheets account
- Google Cloud Console access

### 1. Install n8n
```bash
npm install -g n8n
```

### 2. Start n8n
```bash
N8N_BASIC_AUTH_ACTIVE=false n8n start
```

### 3. Configure Credentials

#### Gmail OAuth2
1. Go to Google Cloud Console
2. Enable Gmail API
3. Create OAuth2 credentials
4. Add credentials in n8n Gmail node

#### Google Sheets OAuth2
1. Go to Google Cloud Console
2. Enable Google Drive API and Google Sheets API
3. Create OAuth2 credentials
4. Add credentials in n8n Google Sheets nodes

### 4. Import Workflow
1. Open n8n at `http://localhost:5678`
2. Import `expense monitor.json`
3. Configure your Google Sheet ID in both Google Sheets nodes
4. Update Gmail filters if needed

### 5. Configure Google Sheet
Create a Google Sheet with the following columns for successful transactions:
- transaction_date
- merchant
- amount
- category
- transaction_type
- description
- extracted_at
- card_number
- reference_number

The failed transactions will be stored with these columns:
- email_id
- subject
- from
- date
- snippet
- failure_reason
- html_body_preview
- extracted_at

## Recent Improvements

### Data Extraction Enhancements
- **Improved Amount Regex**: `/(?:Rs\.?|INR)\s*([\d,]+\.\d{2})/i` catches more amount formats
- **Better Merchant Regex**: `/(?:to|by)\s+([^.]+?)(?:\s+on|\. Your UPI)/i` catches more merchant patterns
- **Enhanced Filtering**: Now passes all transactions with detected types, even if amount is null

### Failed Transaction Tracking
- Captures transactions that fail to process
- Stores failure reasons and email content previews
- Enables debugging of extraction issues

## Troubleshooting

### Common Issues
1. **"No columns found in Google Sheets"**: Ensure your Google Sheet has the correct column headers
2. **Empty transaction details**: Check if emails contain the expected HTML structure
3. **Authentication errors**: Verify OAuth2 credentials are properly configured

### Debugging Failed Transactions
1. Check the "Append Failed Transactions" output in your Google Sheet
2. Review the `failure_reason` and `html_body_preview` columns
3. Adjust regex patterns in the "Process Email Content" node if needed

## File Structure
```
n8n/
├── expense monitor.json          # Main workflow file
├── README.md                     # This documentation
└── Expense_tracker_AI_Agent/     # Additional resources
    ├── README.md
    └── expense monitor.json
```

## Contributing
Feel free to submit issues and enhancement requests!

## License
This project is open source and available under the MIT License. 