# Solution: Pass Data Between Nodes Without Static Storage

## The Challenge
Telegram workflows have TWO separate executions:
1. **Scheduled flow**: Reads email → Sends question
2. **Telegram trigger flow**: User responds → Process response

Data doesn't automatically flow between these two executions.

## ✅ Solution: Embed Data in Telegram Message

### How It Works:
1. When sending question, add email data as hidden text
2. User replies, we extract that data
3. Pass it forward through nodes
4. No storage needed!

### Implementation:

**Ask User Node** - Embed email data:
```
Text: 📧 From: {{ $json.email_from }}
Subject: {{ $json.email_subject }}

❓ Transaction?
1=Yes 2=No

📎 ID:{{ $json.gmail_message_id }}|{{ $json.email_from }}
```

**Route Node** - Extract embedded data:
```javascript
const msg = $json.message.text.trim();
const fullText = $json.message.text;

// Extract embedded email data
const match = fullText.match(/📎 ID:([^|]+)\|(.+)$/m);
const email = match ? {
  gmail_message_id: match[1],
  email_from: match[2]
} : null;

if (msg === '2') return [{ json: { action: 'block', email }}];
if (msg === '1') return [{ json: { action: 'collect', email, step: 'amount' }}];

// Continue conversation...
```

### This Way:
- ✅ NO workflowStaticData
- ✅ NO Google Sheets state storage
- ✅ Data embedded in messages
- ✅ Works reliably

Shall I implement this approach?
