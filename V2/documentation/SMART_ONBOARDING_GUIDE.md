# Smart Onboarding System - Complete Guide

## 🎯 What This Does

The Smart Onboarding system helps you teach your expense tracker about new transaction sources through conversational Telegram interactions. It:

1. **Identifies** unrecognized emails
2. **Asks you** if it's a banking/transaction email
3. **If NO**: Blocks the sender permanently (no more noise)
4. **If YES**: Collects details and creates a provider profile automatically

## 🆕 **How It's Different**

### Old System (CategoryMapper):
- Only categorizes already-parsed transactions
- Requires manual provider setup

### Smart Onboarding System:
- **Handles unrecognized emails** intelligently
- **Creates provider profiles** automatically
- **Blocks spam/non-transaction** emails
- **Learns regex patterns** from your input

## 📋 **Required Google Sheets**

Add one new sheet to your existing document:

### Sheet: "blocked_senders"

Create a new sheet with these columns:

| email_from | blocked_at | reason |
|------------|------------|--------|
| (empty initially) | | |

This will automatically populate when you block senders.

## 🔄 **Complete Workflow**

```
Step 1: Unrecognized Email Arrives
   ↓
Step 2: SmartOnboarding asks via Telegram:
   "Is this a banking email?"
   ↓
┌──────────────┴──────────────┐
│                              │
NO (Block)                    YES (Transaction)
│                              │
↓                              ↓
Add to                      Ask: Amount?
blocked_senders                ↓
  ↓                         Ask: Merchant?
Never see                      ↓
again! ✅                   Ask: Type (DEBIT/CREDIT)?
                               ↓
                            Generate Provider Profile
                               ↓
                            Save to providers sheet
                               ↓
                            Future emails auto-parsed! ✅
```

## 💬 **Conversation Example**

### Scenario: New Email from Amazon Pay

**Bot:**
```
📧 New Email Detected

📨 From: pay-notifications@amazon.in
📋 Subject: Payment successful
📄 Snippet: You have successfully paid Rs. 1,299 to Amazon

❓ Is this a banking/transaction email?

1️⃣ Yes - It's a transaction
2️⃣ No - Block this sender

Reply with 1 or 2
```

**You:** `1`

**Bot:**
```
💰 What is the transaction amount?

Reply with just the number (e.g., 450.50)
```

**You:** `1299`

**Bot:**
```
🏪 What is the merchant/store name?

Reply with the merchant name (e.g., Swiggy, Amazon, Shell)
```

**You:** `Amazon`

**Bot:**
```
💳 Transaction type?

1️⃣ DEBIT (money spent)
2️⃣ CREDIT (money received)

Reply with 1 or 2
```

**You:** `1`

**Bot:**
```
✅ Provider Profile Created!

🆔 Provider ID: amazon_in
📧 Email Filter: pay-notifications@amazon.in
🔍 Amount Regex: (?:INR|Rs\.?|₹)\s*([0-9,]+(?:\.[0-9]{2})?)

📝 Next Steps:
1. Test this provider in next run
2. Refine regex patterns if needed
3. Check providers sheet for details

Future emails from this sender will be parsed automatically!
```

## 🎓 **What Just Happened?**

1. **Provider Created**: New row added to "providers" sheet
2. **Email Filter**: `pay-notifications@amazon.in`
3. **Regex Generated**: Basic pattern for amount extraction
4. **Next Time**: Similar emails from Amazon Pay will be auto-parsed!

## 🚫 **Blocking Non-Transaction Emails**

### Scenario: Newsletter or Marketing Email

**Bot:**
```
📧 New Email Detected

📨 From: newsletter@flipkart.com
📋 Subject: Weekend Sale - Up to 70% off
📄 Snippet: Don't miss our biggest sale...

❓ Is this a banking/transaction email?

1️⃣ Yes - It's a transaction
2️⃣ No - Block this sender

Reply with 1 or 2
```

**You:** `2`

**Bot:**
```
🚫 This sender will be blocked. Future emails will be ignored.
```

**Result:** All future emails from `newsletter@flipkart.com` will be automatically ignored!

## 🔧 **Setup Steps**

### 1. Create "blocked_senders" Sheet

In your Google Sheets:
1. Create new sheet named `blocked_senders`
2. Add columns: `email_from`, `blocked_at`, `reason`
3. Leave it empty (will populate automatically)

### 2. Configure Telegram (if not done already)

Follow the same Telegram setup as Category Mapper:
- Create bot with @BotFather
- Get Chat ID from @userinfobot
- Add Telegram credential in n8n
- Set TELEGRAM_CHAT_ID environment variable

### 3. Import Workflow

1. Import `SmartOnboarding_Telegram.json`
2. Configure Telegram credentials on nodes:
   - "Ask: Is Transaction?"
   - "Telegram Response" trigger
   - "Send Reply (Block)"
   - "Send Reply (Created)"
   - "Send Reply (Other)"
3. Update Chat ID if needed
4. Save workflow

### 4. Test

1. Make sure you have some unrecognized emails
2. Execute the workflow manually
3. Check Telegram for the confirmation message
4. Reply and follow the conversation
5. Verify the result (blocked sender or new provider)

### 5. Activate

1. Toggle workflow to "Active"
2. Runs every 3 hours
3. Processes one unrecognized email at a time

## 📊 **How Profiles Are Generated**

### From Your Input:
- **Email address**: Used for `from_contains` filter
- **Amount**: Used to generate `regex_amount` pattern
- **Merchant**: Used for `regex_merchant` pattern  
- **Type**: Stored in notes for reference

### Basic Regex Patterns Generated:
```
Amount: (?:INR|Rs\.?|₹)\s*([0-9,]+(?:\.[0-9]{2})?)
Merchant: (?:at|to|from)\s+(MerchantName)
```

### You Can Refine Later:
- Open "providers" sheet
- Find the auto-generated row
- Adjust regex patterns based on actual email format
- Add subject_contains or body_contains filters
- Update notes with examples

## 🎯 **Multi-Workflow System**

You now have **3 workflows** working together:

### 1. PersonalExpenseTracker (Every 30 min)
```
Fetches emails → Parses with providers → Stores transactions
                                       → Logs unrecognized
```

### 2. SmartOnboarding (Every 3 hours)
```
Reviews unrecognized → Asks you via Telegram → Blocks OR Creates Provider
```

### 3. CategoryMapper (Every 2 hours)
```
Finds uncategorized → Asks category via Telegram → Learns mappings
```

## ⏱️ **Typical Timeline**

### Day 1:
- **Morning**: New bank emails arrive
- **30 min**: ExpenseTracker processes them
  - Known banks: ✅ Parsed automatically
  - Unknown: ⚠️ Goes to unrecognized
- **3 hours**: SmartOnboarding asks about unrecognized
  - You confirm: Transaction or Block
  - If transaction: Answer 3 questions
  - Provider created!
- **Next run**: Same bank emails now parsed ✅

### After 1 Week:
- Most transaction sources: Auto-parsed ✅
- Spam/newsletters: Blocked 🚫
- Rare merchants: Quick onboarding
- 80%+ automation achieved!

## 🎮 **User Experience**

### For Transaction Sources (One-time Setup):
```
1. Bot shows email snippet
2. You confirm it's a transaction (1 tap)
3. You provide amount (type number)
4. You provide merchant (type name)
5. You select type (1 tap for DEBIT/CREDIT)

Total time: ~30 seconds
Result: Future emails auto-parsed forever!
```

### For Non-Transaction Emails:
```
1. Bot shows email snippet
2. You say "Block" (1 tap)

Total time: 5 seconds
Result: Never see emails from this sender again!
```

## 📈 **Benefits**

### Intelligent Filtering:
- ✅ Learns what's a transaction vs spam
- ✅ Blocks noise permanently
- ✅ Focuses on real expense data

### Self-Improving:
- ✅ Each interaction improves accuracy
- ✅ Provider profiles grow organically
- ✅ Less manual work over time

### Zero Maintenance:
- ✅ No manual provider configuration
- ✅ Regex patterns generated automatically
- ✅ You just answer simple questions

## 🔍 **Monitoring**

### Check Blocked Senders:
Open "blocked_senders" sheet to see all blocked email addresses.

### Check Auto-Generated Providers:
In "providers" sheet, look for rows with notes starting with "Auto-generated from user input"

### Review Unrecognized Queue:
Check "unrecognized_emails" sheet - should decrease over time as you onboard more sources.

## 🆘 **Troubleshooting**

### Bot Not Asking About Unrecognized Emails:
1. Check "unrecognized_emails" sheet has entries
2. Verify they're not from already-blocked senders
3. Check workflow is Active
4. Review execution logs

### Provider Not Working After Creation:
1. Wait for next ExpenseTracker run (30 min)
2. Check if similar email arrives
3. If still unrecognized:
   - Open "providers" sheet
   - Find the auto-generated provider
   - Refine the regex patterns manually
   - Check email snippet for exact format

### Conversation State Lost:
- Telegram uses workflow static data
- Complete the conversation within 1 hour
- If lost, workflow will restart with next unrecognized email

## 🚀 **Advanced: Refining Auto-Generated Providers**

After a provider is auto-generated, you can improve it:

### 1. Get Sample Email:
- Check "unrecognized_emails" for the original email
- Look at the snippet/body format

### 2. Refine Regex:
- Open "providers" sheet
- Find the auto-generated row
- Update `regex_amount` to match exact format
- Example: `Rs\.\s*([0-9,]+\.[0-9]{2})`

### 3. Add More Filters:
- Add `subject_contains` if subject has keywords
- Add `body_contains` if body has specific text
- This makes matching more accurate

### 4. Test:
- Run ExpenseTracker manually
- Check if the email is now parsed correctly

## 🎯 **Quick Reference**

| Action | What Happens | Result |
|--------|--------------|--------|
| Reply "1" (Transaction) | Starts collection flow | Creates provider profile |
| Reply "2" (Block) | Adds to blocklist | Never see this sender again |
| Provide amount | Stored for provider generation | Used in regex pattern |
| Provide merchant | Stored for provider generation | Used in regex pattern |
| Select DEBIT/CREDIT | Completes provider profile | Saved to sheet |

## 📝 **Complete Sheet Structure**

Your Google Sheets now has:

1. **transactions** - All parsed transactions
2. **unrecognized_emails** - Emails needing review
3. **providers** - Parsing rules (some auto-generated!)
4. **blocked_senders** - Permanently ignored senders (NEW)
5. **categories** - Category definitions
6. **merchant_mappings** - Merchant-to-category mappings

## 🎉 **End Result**

After onboarding 10-15 transaction sources:
- ✅ 90%+ emails automatically parsed
- ✅ Spam/newsletters permanently blocked
- ✅ New sources: 30-second onboarding
- ✅ Minimal manual work required

Your expense tracker becomes smarter with every interaction! 🤖🎓
