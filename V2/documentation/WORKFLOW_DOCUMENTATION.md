# Personal Expense Tracker - n8n Workflow Documentation

## Overview
This n8n workflow automatically tracks your expenses by monitoring Gmail for transaction emails, parsing them using configurable provider rules, and storing them in Google Sheets.

## Workflow Structure

```
Schedule Trigger (every 30 min) →
  ├─→ Get Emails (max 100) →
  │   └─→ Dedup (filter processed emails) →
  │       └─→ Merge Emails + Providers →
  └─→ Read Providers Sheet ──────────┘
          │
          └─→ Dynamic Parse Transaction →
              └─→ If Valid Transaction?
                  ├─→ [TRUE] Append to Transactions ✅
                  └─→ [FALSE] Log Unrecognized ⚠️
```

## Nodes Explained

### 1. Schedule Trigger
- **Runs every 30 minutes**
- Triggers the workflow automatically
- Can be adjusted based on your needs

### 2. Get Emails
- Fetches up to 100 emails from Gmail
- Retrieves recent unread or all emails (configurable)
- Returns email metadata: ID, From, Subject, snippet

### 3. Dedup (Performance-Optimized)
- **Uses workflow static data (in-memory storage)**
- Tracks up to 10,000 processed Gmail message IDs
- Prevents duplicate processing of the same email
- **Eliminates need to read Google Sheets for deduplication**
- Maintains a rolling window (keeps 10,000 most recent IDs)

### 4. Read Providers Sheet
- Reads provider rules from Google Sheets "providers" tab
- Each provider defines:
  - Matching filters (from_contains, subject_contains, body_contains)
  - Regex patterns for extracting data (amount, merchant, date, reference)
  - Active status flag

### 5. Merge Emails + Providers
- Combines emails with provider rules
- Passes both datasets to parsing logic

### 6. Dynamic Parse Transaction
- Matches each email against provider rules
- Extracts transaction details using regex:
  - Amount (required)
  - Merchant name
  - Transaction date
  - Reference number
- Marks as valid if amount AND provider_id are successfully extracted

### 7. If Valid Transaction
- **TRUE branch**: Successfully parsed → goes to "Append to Transactions"
- **FALSE branch**: Failed to parse or no matching provider → goes to "Log Unrecognized"

### 8. Append to Transactions
- Writes valid transactions to Google Sheets "transactions" tab
- Includes: timestamp, amount, currency, merchant, provider, email details

### 9. Log Unrecognized
- Writes unparseable emails to "unrecognized_emails" tab
- Helps identify emails that need new provider rules
- Includes provider_guess if a rule partially matched

## Performance Optimizations

### ✅ Removed Slow Operations
- **BEFORE**: Read all existing transactions from Google Sheets every run
- **AFTER**: Use in-memory static data for deduplication
- **Benefit**: 10-100x faster execution, especially with 1000+ transactions

### ✅ In-Memory Deduplication
- Stores Gmail message IDs in workflow static data
- Persists between workflow executions
- Handles up to 10,000 IDs (covers ~1 year of daily transactions)
- Automatically trims oldest IDs when limit reached

### ✅ Single Deduplication Point
- All emails filtered once at the "Dedup" node
- Prevents duplicates in BOTH sheets (transactions & unrecognized)
- No redundant duplicate checking logic

## Google Sheets Structure

### Sheet 1: "providers" (Configuration)
| Column | Description | Example |
|--------|-------------|---------|
| provider_id | Unique identifier | `hdfc_credit_card` |
| active | Enable/disable rule | `TRUE` |
| from_contains | Email sender filter | `alerts@hdfcbank.com` |
| subject_contains | Subject filter | `Credit Card transaction` |
| body_contains | Body content filter | `purchase` |
| regex_amount | Extract amount | `Rs\.?\s*([0-9,]+\.[0-9]{2})` |
| regex_merchant | Extract merchant | `at\s+(.+?)\s+on` |
| regex_date | Extract date | `on\s+(\d{2}-\w{3}-\d{4})` |
| regex_reference | Extract reference | `Ref No:\s*(\w+)` |
| notes | Internal notes | `HDFC alerts format` |

### Sheet 2: "transactions" (Output)
Stores successfully parsed transactions:
- timestamp
- gmail_message_id (unique identifier)
- provider_id
- type (DEBIT/CREDIT)
- amount
- currency
- merchant
- reference
- email_from, email_subject
- raw_body, raw_date

### Sheet 3: "unrecognized_emails" (Review Queue)
Stores emails that couldn't be parsed:
- timestamp
- gmail_message_id
- email_from, email_subject
- snippet_or_body
- provider_guess (if partially matched)

## How to Use

### Initial Setup
1. Import `PersonalExpenseTracker.json` into n8n
2. Configure Google Sheets credentials
3. Configure Gmail credentials
4. Create the three required sheets in Google Sheets:
   - `providers` (with columns as described above)
   - `transactions` (will be populated automatically)
   - `unrecognized_emails` (will be populated automatically)

### Adding Provider Rules
1. Open your Google Sheets document
2. Go to "providers" tab
3. Add a new row with:
   - Unique provider_id
   - Set active = TRUE
   - Add email filters (from/subject/body contains)
   - Add regex patterns for data extraction
   - Test the regex with sample emails

### Testing
1. Click "Execute Workflow" in n8n
2. Check the execution log for:
   - Number of emails fetched
   - Number after deduplication
   - Successful vs. failed parses
3. Review "unrecognized_emails" sheet for any issues

### Monitoring
1. Check "unrecognized_emails" periodically
2. For each unrecognized email:
   - Create a new provider rule if it's a valid transaction source
   - Or ignore if it's not expense-related

### Activating
1. Toggle the workflow to "Active" in n8n
2. It will run automatically every 30 minutes
3. Monitor executions in the n8n UI

## Scaling Considerations

### Current Capacity
- **10,000 message IDs** tracked in memory
- Covers approximately **1 year** of daily transaction emails
- **No performance degradation** as transaction sheet grows

### If You Exceed 10,000 Emails/Year
The workflow automatically maintains a rolling window:
- Keeps the 10,000 most recent message IDs
- Older IDs are dropped
- This is safe because:
  - Gmail typically keeps emails for years
  - You won't re-process very old emails
  - The dedup window covers sufficient history

### Alternative: SQLite Storage (Future Enhancement)
If you want unlimited history tracking, you can:
1. Add an SQLite database node
2. Store message IDs in a table
3. Query before processing
4. This allows tracking millions of IDs efficiently

**However, the current in-memory solution is sufficient for most personal use cases.**

## Troubleshooting

### Email Not Being Captured
1. Check if email is in Gmail
2. Verify Gmail node fetches it (check max results)
3. Check dedup log - might already be processed

### Transaction Not Parsed
1. Check "unrecognized_emails" sheet
2. Review the provider_guess column
3. If provider matched but amount failed:
   - Check the regex_amount pattern
   - Test with actual email content
4. If no provider matched:
   - Add a new provider rule

### Duplicate Transactions
- Should not happen with current dedup logic
- If it does, check if:
  - Workflow was reimported (resets static data)
  - Gmail message IDs are consistent
  
### Performance Issues
- Current workflow should handle 1000s of transactions
- If slow, check:
  - Gmail API rate limits
  - Google Sheets API rate limits
  - Network connectivity

## Maintenance

### Regular Tasks
1. **Weekly**: Review "unrecognized_emails" sheet
2. **Monthly**: Add new provider rules as needed
3. **Quarterly**: Archive old data if sheets become large

### When to Add Provider Rules
- New bank or credit card
- New payment service (UPI, wallet, etc.)
- Transaction email format changes

### Updating Regex Patterns
1. Get a sample email from "unrecognized_emails"
2. Test regex pattern online (regex101.com)
3. Update the provider rule in Google Sheets
4. Test with workflow execution

## Advanced Configuration

### Change Schedule Interval
Edit the Schedule Trigger node:
- Every 15 minutes: `"minutesInterval": 15`
- Every hour: Change to hours interval
- Specific times: Use cron expression

### Change Email Fetch Limit
Edit the Get Emails node:
- Increase `"maxResults": 200` for more emails
- Add filters to only fetch specific labels

### Add Gmail Filters
Edit the Get Emails node filters:
```json
"filters": {
  "labelIds": ["Label_123"],
  "q": "is:unread category:updates"
}
```

### Change Currency
Edit the Dynamic Parse Transaction node:
- Find `parsed.currency = 'INR'`
- Change to your currency code

### Increase Dedup Capacity
Edit the Dedup node:
- Find `const MAX_IDS = 10000`
- Increase to `20000` or higher

## Benefits of This Approach

### ✅ Scalability
- Handles unlimited transactions in sheets
- No slowdown as data grows
- Memory-efficient deduplication

### ✅ Reliability
- Prevents duplicate processing
- Tracks all failures for review
- Robust error handling

### ✅ Flexibility
- Easy to add new providers
- Regex-based extraction
- Customizable schedules

### ✅ Visibility
- Logs all unrecognized emails
- Clear transaction history
- Easy troubleshooting

## Support & Improvements

### Future Enhancements
- [ ] SQLite integration for unlimited ID storage
- [ ] Email body fetch for better parsing
- [ ] Auto-categorization with AI
- [ ] Monthly summary reports
- [ ] Budget alerts
- [ ] Multi-currency support

### Questions or Issues?
Review the n8n execution logs and check:
- Node outputs at each step
- Error messages in failed executions
- Console logs in Code nodes
