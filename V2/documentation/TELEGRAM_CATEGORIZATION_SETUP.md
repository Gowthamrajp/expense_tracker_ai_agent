# Telegram Categorization Setup Guide

## 🎯 Overview

This system allows you to categorize expenses via Telegram bot interactions. It learns from your inputs and auto-categorizes future transactions from known merchants.

## 📋 **Required Google Sheets Structure**

You need to add 2 new sheets to your existing Google Sheets document:

### Sheet 1: "categories"

Create a new sheet named "categories" with these columns:

| category | subcategory | icon | notes |
|----------|-------------|------|-------|
| Food | Groceries | 🛒 | Supermarkets, vegetables |
| Food | Restaurants | 🍽️ | Dining out, food delivery |
| Food | Coffee & Snacks | ☕ | Cafes, quick bites |
| Transport | Fuel | ⛽ | Petrol, diesel |
| Transport | Public Transport | 🚌 | Metro, bus, auto |
| Transport | Ride-sharing | 🚕 | Uber, Ola |
| Shopping | Clothing | 👕 | Clothes, shoes |
| Shopping | Electronics | 📱 | Gadgets, accessories |
| Shopping | Home & Living | 🏠 | Furniture, decor |
| Bills & Utilities | Electricity | 💡 | Power bills |
| Bills & Utilities | Internet | 🌐 | Broadband, data |
| Bills & Utilities | Mobile | 📱 | Phone recharge |
| Entertainment | Movies & Shows | 🎬 | Cinema, streaming |
| Entertainment | Events | 🎉 | Concerts, activities |
| Health | Medical | 💊 | Doctor, pharmacy |
| Health | Fitness | 💪 | Gym, sports |
| Personal Care | Grooming | ✂️ | Salon, spa |
| Personal Care | Beauty | 💄 | Cosmetics |
| Other | Miscellaneous | 📦 | Uncategorized |

**Add more categories as needed!**

### Sheet 2: "merchant_mappings"

Create a new sheet named "merchant_mappings" with these columns:

| merchant_key | category | subcategory | learned_at | notes |
|--------------|----------|-------------|------------|-------|
| (empty initially - will be populated automatically) |

This sheet will automatically populate as you categorize transactions via Telegram.

### Update "transactions" Sheet

Add these columns to your existing "transactions" sheet:
- **category** (after "merchant" column)
- **subcategory** (after "category" column)

Your transactions sheet should now have:
`timestamp, gmail_message_id, provider_id, type, amount, currency, merchant, category, subcategory, email_from, email_subject, raw_date, raw_body, reference`

## 🤖 **Telegram Bot Setup**

### Step 1: Create Telegram Bot

1. Open Telegram and search for `@BotFather`
2. Send `/newbot`
3. Follow prompts:
   - Choose a name (e.g., "My Expense Categorizer")
   - Choose a username (e.g., "my_expense_cat_bot")
4. **Save the API Token** (looks like: `123456789:ABCdefGHIjklMNOpqrsTUVwxyz`)

### Step 2: Get Your Chat ID

1. Search for `@userinfobot` in Telegram
2. Start a chat - it will show your Chat ID
3. **Save your Chat ID** (looks like: `123456789`)

### Step 3: Configure n8n

1. In n8n, go to **Settings → Credentials**
2. Click **"+ New Credential"**
3. Search for and select **"Telegram API"**
4. Configure:
   - **Name**: `Telegram Expense Bot`
   - **Access Token**: (paste your Bot Token from BotFather)
5. Click **"Save"**

### Step 4: Set Environment Variable

Add your Chat ID as an environment variable:

**If using Docker**, restart with this additional env var:
```bash
-e TELEGRAM_CHAT_ID=YOUR_CHAT_ID_HERE
```

**Or** edit the workflow and replace `{{ $env.TELEGRAM_CHAT_ID }}` with your actual chat ID.

## 📥 **Import CategoryMapper_Telegram.json**

1. Import `CategoryMapper_Telegram.json` into n8n
2. **Configure Telegram credentials** on these nodes:
   - "Send to Telegram" node
   - "Telegram Response" trigger
   - "Confirm via Telegram" node
3. Update Chat ID if not using environment variable
4. **Save** the workflow

## 🔄 **How It Works**

### Workflow Flow:

```
Schedule (every 2 hours) →
  ├─→ Read Uncategorized Transactions (where category is empty)
  ├─→ Read Categories (predefined categories)
  └─→ Read Merchant Mappings (learned mappings)
      │
      └─→ Auto Categorize
          ├─→ [Known Merchant] Auto-apply category
          └─→ [Unknown Merchant] Ask user via Telegram
              │
              ↓
          User Replies via Telegram
              │
              └─→ Update Transaction + Save Mapping
```

### User Interaction Example:

**Bot sends:**
```
🏷️ New Transaction Needs Categorization

💰 Amount: ₹450
🏪 Merchant: Swiggy
📅 Date: 2025-11-26

Available Categories:

1. Food
   2. Food > Groceries
   3. Food > Restaurants
   4. Food > Coffee & Snacks

5. Transport
   6. Transport > Fuel
   7. Transport > Public Transport
   8. Transport > Ride-sharing

9. Shopping
   10. Shopping > Clothing
   11. Shopping > Electronics

...

25. ➕ Add New Category

Reply with the number of your choice.
```

**You reply:** `3`

**Bot confirms:**
```
✅ Categorized as: Food > Restaurants

This merchant will be auto-categorized next time!
```

## 🎓 **Learning System**

### First Time (Manual):
1. Bot asks you to categorize "Swiggy"
2. You choose "Food > Restaurants"
3. Mapping saved: `Swiggy → Food > Restaurants`

### Next Time (Automatic):
1. Bot sees "Swiggy" transaction
2. Auto-applies "Food > Restaurants"
3. No user interaction needed! ✅

## ➕ **Adding New Categories**

### Via Telegram:
1. When asked to categorize, reply with the "Add New Category" option number
2. Bot will ask for new category name
3. Type the name (e.g., "Education")
4. Bot adds it to the categories sheet
5. Next transaction will show this new category

### Via Google Sheets:
1. Open your Google Sheets
2. Go to "categories" tab
3. Add a new row:
   - category: "Education"
   - subcategory: "Books"
   - icon: "📚"
   - notes: "Textbooks, courses"
4. Next categorization request will include this option

## 🔧 **Configuration Options**

### Change Schedule Frequency

Edit the Schedule Trigger node:
- **Every hour**: `"hoursInterval": 1`
- **Every 4 hours**: `"hoursInterval": 4`
- **Daily at 9 PM**: Use cron expression `0 21 * * *`

### Process Multiple Transactions

Currently processes one at a time to avoid overwhelming you. To process more:

Edit "Get First Transaction" node to take more items:
```javascript
// Take first 5 uncategorized transactions
return items.slice(0, 5);
```

## 📊 **Google Sheets Updates**

### Transactions Sheet (Updated Schema):

| Column | Description | Example |
|--------|-------------|---------|
| timestamp | When captured | 2025-11-26... |
| gmail_message_id | Unique ID | abc123xyz |
| provider_id | Bank/Card | hdfc_credit |
| type | DEBIT/CREDIT | DEBIT |
| amount | Transaction amount | 450.00 |
| currency | Currency code | INR |
| merchant | Merchant name | Swiggy |
| **category** | **Main category** | **Food** |
| **subcategory** | **Specific type** | **Restaurants** |
| email_from | Source email | alerts@hdfc |
| email_subject | Email subject | Transaction alert |
| raw_date | Original date | 26-Nov-2025 |
| raw_body | Email snippet | Purchase at Swiggy |
| reference | Transaction ref | REF12345 |

### Categories Sheet (Pre-populated):

| category | subcategory | icon | notes |
|----------|-------------|------|-------|
| Food | Groceries | 🛒 | Supermarkets |
| Food | Restaurants | 🍽️ | Dining |
| Transport | Fuel | ⛽ | Petrol |
| ... | ... | ... | ... |

### Merchant Mappings Sheet (Auto-populated):

| merchant_key | category | subcategory | learned_at | notes |
|--------------|----------|-------------|------------|-------|
| swiggy | Food | Restaurants | 2025-11-26... | Auto-learned |
| zomato | Food | Restaurants | 2025-11-26... | Auto-learned |
| shell | Transport | Fuel | 2025-11-26... | Auto-learned |

## 🎯 **Usage Workflow**

### Daily Routine:

1. **Morning**: Expense Tracker runs, captures new transactions
2. **Every 2 hours**: Category Mapper checks for uncategorized transactions
3. **If found**: Sends Telegram message asking for category
4. **You reply**: Choose from numbered options (takes 5 seconds)
5. **Bot learns**: Saves mapping for future auto-categorization
6. **Over time**: Less manual work as more merchants are mapped!

### After 1 Month:
- Most common merchants: Auto-categorized ✅
- Rare/new merchants: Quick Telegram categorization
- 80%+ transactions categorized automatically

## 📈 **Benefits**

### Smart Learning:
- ✅ One-time manual categorization per merchant
- ✅ Future transactions auto-categorized
- ✅ Builds your personal merchant database

### Flexible:
- ✅ Add new categories anytime
- ✅ Update existing mappings
- ✅ Custom categories for your spending habits

### Mobile-Friendly:
- ✅ Categorize from anywhere via Telegram
- ✅ Quick interaction (5 seconds per transaction)
- ✅ Immediate confirmation

## 🔍 **Monitoring**

### Check Auto-Categorization Rate:

```sql
-- Via Google Sheets formula in a summary cell
=COUNTA(FILTER(transactions!category:category, transactions!category:category<>""))
  / COUNTA(transactions!amount:amount)
```

### View Learned Mappings:

Open "merchant_mappings" sheet to see all learned merchant-to-category associations.

## 🆘 **Troubleshooting**

### Bot Not Responding:
1. Check Telegram credential is configured
2. Verify Chat ID is correct
3. Check workflow is Active
4. Review n8n execution logs

### Categories Not Showing:
1. Verify "categories" sheet exists
2. Check sheet name is exactly "categories" (lowercase)
3. Ensure columns are: category, subcategory, icon, notes

### Merchant Mapping Not Saving:
1. Verify "merchant_mappings" sheet exists
2. Check merchant name is not empty
3. Review node execution output

## 🚀 **Future Enhancements**

### Advanced Features (Optional):
- [ ] AI-powered merchant name normalization
- [ ] Spending insights via Telegram (`/summary` command)
- [ ] Budget alerts when category limits exceeded
- [ ] Monthly reports sent via Telegram
- [ ] Category-wise spending charts
- [ ] Split transactions (multiple categories)

Your categorization system is ready to learn! 🎓
