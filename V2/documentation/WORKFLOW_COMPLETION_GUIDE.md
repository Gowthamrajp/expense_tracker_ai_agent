# SmartOnboarding Workflow - Completion Guide

## ✅ What's Working

- **Scheduled trigger** - Runs every hour
- **Read Unrecognized** - Fetches unrecognized emails
- **Ask** - Sends question to Telegram
- **Block flow** (Reply "2"):
  - User Response trigger
  - Is 2? IF node
  - Read Latest Email
  - Block (append to blocked_senders)
  - Confirm Block
- **Transaction start** (Reply "1"):
  - User Response trigger
  - Is 2? IF node (false path)
  - Is 1? IF node
  - Ask Amount

## ❌ What's Missing

After "Ask Amount", the workflow is incomplete. You need to add:

1. **Collect Amount** - User replies with number
2. **Ask for Merchant** - System asks "🏪 Merchant?"
3. **Collect Merchant** - User replies with merchant name
4. **Ask for Type** - System asks "💳 1=DEBIT 2=CREDIT?"
5. **Collect Type** - User replies 1 or 2
6. **Generate Prompt** - Create regex pattern prompt
7. **Send Prompt** - Send to user
8. **Handle DONE** - Backfill similar emails

## 🚨 The Problem

**Code nodes hang forever in your n8n instance.** This makes building complex state management very difficult.

## 💡 Recommended Solutions

### Option 1: Manual Completion in n8n UI (Recommended)

The best way to complete this workflow is manually in the n8n UI using the visual editor:

1. **For each subsequent response**, you need to:
   - Add a new Telegram Trigger (with different webhook)
   - Add IF nodes to detect responses
   - Store data using Set nodes or Google Sheets
   - Ask next question

2. **Use sticky notes** in n8n to document each step

3. **Test incrementally** - Add one step at a time

### Option 2: Use Existing Working Workflow

Your original `SmartOnboarding_Telegram.json` (before all my changes) was actually working with Code nodes. The issue was trying to add inline keyboards.

**Recommendation:** Revert to your original text-based workflow that was working.

### Option 3: Simplified Approach

**Just handle blocking for now:**
- Current workflow blocks senders ✅
- For transactions, manually process them (don't automate onboarding yet)
- Focus on getting the block functionality stable first

## 📝 Complete Flow Architecture

If you want to build the complete flow without Code nodes:

```
User Response
  ↓
Is message "2"?
  ├─ Yes → Read Latest Email → Block → Confirm
  └─ No → Is message "1"?
            ├─ Yes → Ask Amount
            └─ No → Is numeric?
                      ├─ Yes → Store Amount → Ask Merchant
                      └─ No → Is "DONE"?
                                ├─ Yes → Backfill
                                └─ No → Store Merchant → Ask Type
```

Each step requires:
- Multiple Telegram Triggers (one per interaction stage)
- IF nodes for routing
- Google Sheets for storing state
- NO Code nodes (they hang)

## 🎯 My Recommendation

**Use your original working workflow** before trying to add inline keyboards. The text-based version with Code nodes was functional.

The current simplified version works for:
✅ Scheduled hourly checks
✅ Blocking senders

But completing the full onboarding flow without Code nodes is complex and better done manually in the n8n visual editor.

## 📚 Reference

See the original working logic in your backed-up workflows or version control.

The key insight: **Code nodes hang in your n8n instance** - this is likely an environment issue (memory, CPU, or n8n version bug).

Consider:
1. Updating n8n to latest version
2. Checking system resources
3. Reviewing n8n logs for errors
4. Testing Code nodes in isolation

Would you like me to restore your original working workflow instead?
