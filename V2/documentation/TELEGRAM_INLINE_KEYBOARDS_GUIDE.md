# Telegram Inline Keyboards Guide

## Overview
The SmartOnboarding workflow now uses **Telegram Inline Keyboards** for a much better user experience. Instead of typing numbers, you can simply tap buttons!

## 🎯 What Changed

### ✅ Before (Text-Based)
```
📧 From: sender@example.com
Subject: Payment received

❓ Transaction?
1=Yes 2=No

🔖message_id|from|subject|body
```
User had to type: `1` or `2`

### 🎉 After (Interactive Buttons)
```
📧 From: sender@example.com
Subject: Payment received

❓ Is this a transaction?

[✅ Yes]  [❌ No (Block)]
```
User just taps a button!

## 🔧 Technical Implementation

### Two Telegram Triggers
The workflow now has **two triggers** to handle different input types:

1. **User Text** - Handles text messages (amounts, merchant names, DONE command)
2. **User Buttons** - Handles inline keyboard button clicks (callback queries)

### Button Data Format
Buttons send callback data in this format:
```
action|message_id|email_from|email_subject|body
```

Examples:
- `yes|ABC123|sender@example.com|Payment|Body text`
- `block|ABC123|sender@example.com|Spam|Unwanted`
- `debit|Amazon`
- `credit|Salary`

## 🎨 Interactive Questions

### 1️⃣ Initial Question: Is this a transaction?
**Buttons:**
- ✅ Yes → Asks for amount
- ❌ No (Block) → Blocks sender immediately

**Implementation:**
```javascript
"inlineKeyboard": {
  "inline_keyboard": [[
    {"text": "✅ Yes", "callback_data": "yes|..."},
    {"text": "❌ No (Block)", "callback_data": "block|..."}
  ]]
}
```

### 2️⃣ Transaction Type: Debit or Credit?
**Buttons:**
- 💸 Debit → Records as DEBIT
- 💰 Credit → Records as CREDIT

**Implementation:**
```javascript
"inlineKeyboard": {
  "inline_keyboard": [[
    {"text": "💸 Debit", "callback_data": "debit|merchant"},
    {"text": "💰 Credit", "callback_data": "credit|merchant"}
  ]]
}
```

## 🔄 Workflow Flow

### Scheduled Flow (Every Hour)
```
Every Hour
    ↓
Read Unrecognized Emails
    ↓
Limit (process one at a time)
    ↓
Ask (with Yes/No buttons)
    ↓
⏳ Wait for user response...
```

### User Clicks "✅ Yes"
```
User Buttons Trigger
    ↓
Route Buttons (parses callback)
    ↓
Ask Amount? (text input)
    ↓
User Text Trigger (types amount)
    ↓
Route Text
    ↓
Ask Merchant? (text input)
    ↓
User types merchant name
    ↓
Ask Type? (Debit/Credit buttons)
    ↓
User clicks button
    ↓
Generate regex prompt
```

### User Clicks "❌ No (Block)"
```
User Buttons Trigger
    ↓
Route Buttons
    ↓
Block? → Yes
    ↓
Add to blocked_senders sheet
    ↓
Confirm Block (shows "🚫 Blocked sender")
```

## 💡 Mixed Input Strategy

The workflow intelligently uses:
- **Buttons** for simple Yes/No or multiple choice → Better UX
- **Text input** for amounts and names → More flexible

### When to Use Buttons
✅ Yes/No questions  
✅ Fixed options (Debit/Credit)  
✅ Quick selections  

### When to Use Text
✅ Numeric input (amounts)  
✅ Free-form text (merchant names)  
✅ Commands (DONE)  

## 🎯 Benefits

### 1. **Faster Response**
- No typing required for Yes/No
- One tap vs typing "1" or "2"

### 2. **Fewer Errors**
- Can't type wrong number
- Buttons are self-explanatory

### 3. **Better Mobile Experience**
- Tap-friendly interface
- No need to switch to keyboard

### 4. **Visual Clarity**
- Emojis make options clear
- Buttons stand out visually

## 🔧 Callback Query Handling

### What is a Callback Query?
When a user clicks an inline keyboard button, Telegram sends a **callback query** instead of a regular message.

### Key Differences
| Feature | Text Message | Callback Query |
|---------|-------------|----------------|
| Trigger | `message` | `callback_query` |
| Data location | `message.text` | `callback_query.data` |
| Chat ID | `message.chat.id` | `callback_query.message.chat.id` |
| Callback ID | N/A | `callback_query.id` (for acknowledgment) |

### Answering Callbacks
Always acknowledge button clicks with:
```javascript
operation: "answerInlineQuery"
queryId: "={{ $json.callback_id }}"
text: "Confirmation message"
```

This makes the button feel responsive!

## 📝 Adding More Buttons

Want to add more button options? Here's how:

### Example: Add "Skip" Button
```javascript
"inlineKeyboard": {
  "inline_keyboard": [[
    {"text": "✅ Yes", "callback_data": "yes|..."},
    {"text": "❌ No (Block)", "callback_data": "block|..."},
    {"text": "⏭️ Skip", "callback_data": "skip|..."}  // New!
  ]]
}
```

Then handle in Route Buttons:
```javascript
if (action === 'skip') {
  return { json: { action: 'skip', chatId, email, callback_id }};
}
```

### Example: Add Category Buttons
```javascript
"inlineKeyboard": {
  "inline_keyboard": [
    [{"text": "🍔 Food", "callback_data": "category|food"}],
    [{"text": "🚗 Transport", "callback_data": "category|transport"}],
    [{"text": "🏠 Bills", "callback_data": "category|bills"}]
  ]
}
```

## 🚀 Next Steps

### Potential Enhancements
1. **Amount Presets**: Add buttons for common amounts ($10, $50, $100)
2. **Merchant Quick Select**: Show recent merchants as buttons
3. **Category Selection**: Add category buttons instead of text input
4. **Confirmation Step**: Show summary with Confirm/Edit buttons

### Example: Amount Presets
```javascript
"inlineKeyboard": {
  "inline_keyboard": [
    [
      {"text": "💵 $10", "callback_data": "amount|10"},
      {"text": "💵 $20", "callback_data": "amount|20"},
      {"text": "💵 $50", "callback_data": "amount|50"}
    ],
    [{"text": "✏️ Custom Amount", "callback_data": "amount|custom"}]
  ]
}
```

## 🐛 Troubleshooting

### Buttons Not Appearing
- Check that `replyMarkup` is set to `"inlineKeyboard"`
- Verify JSON syntax in `inlineKeyboard` parameter
- Ensure proper n8n expression syntax with `=`

### Callback Not Received
- Confirm `User Buttons` trigger has `callback_query` in updates
- Check webhook is properly registered
- Verify Telegram API credentials

### Button Click Has No Effect
- Ensure `Route Buttons` node is connected
- Check callback_data parsing logic
- Verify action routing conditions

## 📚 Resources

- [Telegram Bot API - Inline Keyboards](https://core.telegram.org/bots/api#inlinekeyboardmarkup)
- [n8n Telegram Node Documentation](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.telegram/)
- [Telegram Bot Best Practices](https://core.telegram.org/bots/features#inline-keyboards)

## ✨ Summary

The inline keyboard implementation transforms the onboarding experience from:
- **Text-based** (type numbers) → **Visual** (tap buttons)
- **Error-prone** (typos) → **Error-free** (predefined actions)
- **Slow** (type, send) → **Fast** (single tap)

This creates a **modern, mobile-first user experience** that's perfect for managing transactions on the go! 🎉
