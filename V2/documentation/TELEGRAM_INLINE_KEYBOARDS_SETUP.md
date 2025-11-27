# Telegram Inline Keyboards - Quick Setup Guide

## ✅ Fixed Issues

### 1. **Single Telegram Trigger** (CRITICAL FIX)
- ❌ Before: Two separate triggers (User Text + User Buttons) → **Telegram API error**
- ✅ After: One unified trigger (User Response) → Works perfectly!

**Why?** Telegram only allows ONE trigger per bot. Having two triggers causes the error you saw:
> "Due to Telegram API limitations, you can use just one Telegram trigger for each bot at a time"

### 2. **Unified Routing Logic**
- Single "Route" node handles BOTH button clicks AND text messages
- Automatically detects if response is a button click or text input
- Routes to appropriate next step

### 3. **Simplified Callback Data**
- Only passes `gmail_message_id` in callback_data (not full email)
- Extracts email details from message text itself
- Stays under Telegram's 64-byte callback_data limit

## 🚀 How to Use

### Step 1: Import Workflow
```
1. Open n8n
2. Deactivate/delete old "SmartOnboarding_Telegram" workflow
3. Import → Select "SmartOnboarding_Telegram.json"
4. Click "Import"
```

### Step 2: Activate
```
Click the "Active" toggle (should turn blue/green)
```

### Step 3: Test
```
Option A: Wait for next hour
Option B: Click "Every Hour" node → "Execute Node" (manual trigger)
```

### Step 4: Check Telegram
You should now see buttons! The message will look like:
```
📧 From: Medium Daily Digest <noreply@medium.com>
Subject: 1984 Love byte...

❓ Is this a transaction?

🔖 19abdf5ed1a68cbb

[✅ Yes]  [❌ No (Block)]  ← Tap these buttons!
```

## 📱 User Experience Flow

### Scenario 1: It's a Transaction
```
1. Message arrives with buttons
2. Tap [✅ Yes]
3. System: "💰 Enter the amount:"
4. Type: 2500
5. System: "🏪 Enter merchant/vendor name:"
6. Type: Amazon
7. System shows buttons: "💳 Transaction type:"
   [💸 Debit]  [💰 Credit]
8. Tap [💸 Debit]
9. System generates regex prompt
```

### Scenario 2: Not a Transaction (Block)
```
1. Message arrives with buttons
2. Tap [❌ No (Block)]
3. System: "🚫 Blocked sender" (instant feedback)
4. Sender added to blocked_senders sheet
```

### Scenario 3: Backfill Existing Emails
```
1. Complete flow for one email (get to regex prompt)
2. Add regex to providers sheet
3. Reply: DONE
4. System processes all emails from same sender automatically
```

## 🎯 Key Components

### Single Telegram Trigger: "User Response"
```javascript
Updates: ["message", "callback_query"]
```
This ONE trigger handles:
- Text messages (amounts, merchant names, DONE)
- Button clicks (Yes/No, Debit/Credit)

### Unified Route Node
```javascript
if (isCallback) {
  // Handle button clicks
  // Extract email data from message text
} else {
  // Handle text input
  // Parse amount, merchant, or DONE command
}
```

### Interactive Questions
1. **Ask** - Yes/No buttons for transaction confirmation
2. **Ask Amount** - Text input for amount
3. **Ask Merchant** - Text input for merchant
4. **Ask Type** - Debit/Credit buttons

## 🔍 Troubleshooting

### If Buttons Still Don't Appear

**Check "Ask" Node Configuration:**
1. Open "Ask" node
2. Go to "Additional Fields"
3. Verify:
   - Reply Markup: `inlineKeyboard` (from dropdown)
   - Inline Keyboard: Expression starting with `=`
   ```json
   ={"inline_keyboard":[[{"text":"✅ Yes","callback_data":"yes|{{ $json.gmail_message_id }}"},{"text":"❌ No (Block)","callback_data":"block|{{ $json.gmail_message_id }}"}]]}
   ```

**Common Mistakes:**
- ❌ Missing `=` at start of expression
- ❌ Wrong Reply Markup selection
- ❌ Malformed JSON in inline keyboard

### If "User Response" Trigger Shows Error

This usually means:
1. Another workflow is using the same Telegram bot
2. Old webhook still registered

**Fix:**
1. Deactivate ALL workflows using this Telegram bot
2. Wait 30 seconds
3. Activate only the new workflow
4. This re-registers the webhook correctly

## 📊 Data Flow

```
Sheet "unrecognized_emails"
    ↓
Schedule Trigger (Every Hour)
    ↓
Read Unrecognized
    ↓
Limit (1 at a time)
    ↓
Ask (with Yes/No buttons)
    ↓
⏳ User clicks button when free
    ↓
User Response Trigger
    ↓
Route (determines next step)
    ↓
Various actions based on choice
```

## 💡 Benefits

✅ **No Multiple Trigger Errors** - Single trigger handles everything  
✅ **Interactive Buttons** - Tap instead of type for Yes/No  
✅ **Works Asynchronously** - Respond when you're free  
✅ **Smart Routing** - Automatically handles buttons vs text  
✅ **Mobile Friendly** - Perfect for on-the-go responses  

## 🎉 Ready to Use!

The workflow is now configured correctly with:
- ✅ Single Telegram trigger (no API conflicts)
- ✅ Inline keyboards for Yes/No and Debit/Credit
- ✅ Text input for amounts and merchant names
- ✅ Unified routing logic
- ✅ All connections properly wired

Just import, activate, and test! The next message will have buttons. 🚀
