# Telegram Inline Buttons - Troubleshooting Guide

## ❓ Why No Buttons Appeared

If you received a message without buttons, here are the common reasons:

### 1. **Message Already Sent** (Most Common)
- The message you received was sent with the OLD workflow version
- Buttons only appear on NEW messages sent after updating the workflow
- **Solution**: Wait for the next scheduled run (hourly) to see buttons on new emails

### 2. **Workflow Not Re-imported**
- The updated JSON file needs to be imported into n8n
- **Solution**: Delete old workflow and import the new `SmartOnboarding_Telegram.json`

### 3. **Workflow Not Activated**
- The workflow must be active to send new messages
- **Solution**: Click the "Active" toggle in n8n to activate

## ✅ Quick Fix Steps

### Step 1: Re-import Updated Workflow
1. Open n8n
2. Go to Workflows
3. Delete the old "SmartOnboarding_Telegram" workflow (or rename it)
4. Click "+ Add Workflow" → "Import from File"
5. Select `SmartOnboarding_Telegram.json`

### Step 2: Verify Telegram Credentials
1. Open the workflow
2. Check any Telegram node (Ask, User Text, User Buttons)
3. Ensure "Telegram account" credential is selected
4. Should show: `KPVbdDwhR4AzKcA5`

### Step 3: Activate Workflow
1. Click the "Active" toggle (top right)
2. Should turn green/blue when active

### Step 4: Test Immediately (Optional)
Instead of waiting for hourly trigger:
1. Click on "Every Hour" node
2. Click "Execute Node" button
3. This triggers the workflow immediately
4. Check Telegram for new message with buttons

## 🔍 Verify Inline Keyboard Configuration

### Check "Ask" Node
Open the "Ask" node and verify these settings:

**Text field should show:**
```
=📧 From: {{ $json.email_from }}
Subject: {{ $json.email_subject }}

❓ Is this a transaction?

🔖 {{ $json.gmail_message_id }}
```

**Additional Fields:**
- Reply Markup: `inlineKeyboard` (dropdown)
- Inline Keyboard (expression): 
```json
={"inline_keyboard":[[{"text":"✅ Yes","callback_data":"yes|{{ $json.gmail_message_id }}"},{"text":"❌ No (Block)","callback_data":"block|{{ $json.gmail_message_id }}"}]]}
```

⚠️ **Important**: The expression must start with `=` symbol!

### Check "Ask Type" Node
**Additional Fields:**
- Reply Markup: `inlineKeyboard`
- Inline Keyboard (expression):
```json
={
  "inline_keyboard": [
    [
      {"text": "💸 Debit", "callback_data": "debit|{{ $json.merchant }}"},
      {"text": "💰 Credit", "callback_data": "credit|{{ $json.merchant }}"}
    ]
  ]
}
```

## 🧪 Test Inline Keyboards

### Manual Test
1. Open "Ask" node
2. Click "Execute Node"
3. Check if execution succeeds
4. Look for error messages in execution log
5. Check Telegram for the message

### Expected Result
You should see:
```
📧 From: sender@example.com
Subject: Transaction alert

❓ Is this a transaction?

🔖 message_id_123

[✅ Yes]  [❌ No (Block)]  ← These are clickable buttons!
```

### What Buttons Look Like
- **Desktop Telegram**: Blue/gray rounded buttons below message
- **Mobile Telegram**: Buttons appear inline below the message text
- **Telegram Web**: Similar to desktop, buttons below message

## 🔧 Common Issues & Fixes

### Issue: "Invalid JSON in inlineKeyboard"
**Cause**: Malformed JSON expression  
**Fix**: Ensure proper quotes and no extra spaces in JSON

### Issue: Buttons appear but clicking does nothing
**Cause**: `User Buttons` trigger not active or webhook not registered  
**Fix**: 
1. Check `User Buttons` node has `callback_query` in updates
2. Deactivate and reactivate workflow to re-register webhook

### Issue: Getting "callback_query not found" error
**Cause**: Route Buttons node expecting wrong structure  
**Fix**: The updated code now extracts data from message text, should work correctly

## 📱 How to Test

### Test Scenario 1: Complete Happy Path
1. Wait for scheduled message (or manually trigger)
2. You'll receive message with `[✅ Yes]  [❌ No (Block)]` buttons
3. Click `✅ Yes`
4. System asks: "💰 Enter the amount:"
5. Type: `25.50`
6. System asks: "🏪 Enter merchant/vendor name:"
7. Type: `Amazon`
8. System asks: "💳 Transaction type:" with `[💸 Debit]  [💰 Credit]` buttons
9. Click `💸 Debit`
10. System generates regex prompt

### Test Scenario 2: Block Sender
1. Receive message with buttons
2. Click `❌ No (Block)`
3. System shows: "🚫 Blocked sender"
4. Sender added to blocked_senders sheet

## 🎯 Callback Data Limits

Telegram has a **64-byte limit** for callback_data. The updated workflow uses minimal data:
- ✅ `yes|message_id` (short, fits easily)
- ✅ `block|message_id` (short, fits easily)
- ✅ `debit|merchant` (short if merchant name is reasonable)
- ✅ `credit|merchant` (short if merchant name is reasonable)

Email details (from, subject, body) are extracted from the message text itself, not passed in callback_data.

## 🚀 Next Message Will Have Buttons

**Important**: The message you already received won't magically get buttons added. You need to:
1. Import the updated workflow
2. Activate it
3. Wait for the NEXT scheduled email (or trigger manually)
4. The NEW message will have buttons

## 📞 Still Not Working?

Check execution logs:
1. Go to "Executions" tab in n8n
2. Find the latest execution
3. Look for error messages in "Ask" node
4. Common errors:
   - Missing credentials
   - Invalid JSON syntax
   - Telegram API rate limits
   - Network connectivity issues

## ✅ Success Indicators

When working correctly, you'll see:
- ✅ Buttons appear below message
- ✅ Clicking button gives instant feedback (loading animation)
- ✅ Workflow continues to next step
- ✅ No need to type numbers anymore!

The workflow is ready - just needs to send a NEW message with the updated configuration! 🎉
