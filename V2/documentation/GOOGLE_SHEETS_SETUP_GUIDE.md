# Google Sheets Setup Guide - Complete Instructions

## 🎯 Overview

Your current transactions sheet has Gmail metadata columns. We'll **keep those** and just **add the missing columns** for the new system to work.

## 📊 **Sheet 1: transactions (UPDATE EXISTING)**

### Current Columns (Keep These):
Your existing columns: `id, threadId, snippet, payload, sizeEstimate, historyId, internalDate, labels, To, Subject, From, parsed`

### Add These New Columns (After "parsed"):

Right-click on column after "parsed" → Insert columns → Add these headers:

| provider_id | type | amount | currency | merchant | category | subcategory | raw_date | reference |

### Instructions:
1. Open your Google Sheets
2. Go to "transactions" sheet
3. Click on the column header after "parsed" (column M)
4. Right-click → Insert 9 columns to the right
5. Add these headers in the new columns:
   - Column N: `provider_id`
   - Column O: `type`
   - Column P: `amount`
   - Column Q: `currency`
   - Column R: `merchant`
   - Column S: `category`
   - Column T: `subcategory`
   - Column U: `raw_date`
   - Column V: `reference`

### Result:
Your transactions sheet will have both old Gmail data AND new transaction data columns. The workflow will populate the new columns.

---

## 📊 **Sheet 2: providers (Keep As-Is or Create)**

If you already have this sheet, great! If not, create it:

### Headers:
| provider_id | active | from_contains | subject_contains | body_contains | regex_amount | regex_merchant | regex_date | regex_reference | notes |

### Sample Rows (Add these if starting fresh):

**HDFC UPI Savings:**
| HDFC_UPI_SAV | TRUE | alerts@hdfcbank.net | UPI txn | debited from account | Rs\\.([0-9,]+\\.[0-9]{2})\\s+has been debited from account | to VPA\\s+\\S+\\s+(.+?)\\s+on | on\\s+([0-9]{2}-[0-9]{2}-[0-9]{2}) | reference number is\\s+(\\d+) | HDFC Savings UPI |

**HDFC UPI Credit Card:**
| HDFC_UPI_CC | TRUE | alerts@hdfcbank.net | UPI txn | debited from your HDFC Bank | Rs\\.([0-9,]+\\.[0-9]{2})\\s+has been debited from your | to\\s+\\S+\\s+(.+?)\\s+on | on\\s+([0-9]{2}-[0-9]{2}-[0-9]{2}) | reference number is\\s+(\\d+) | HDFC Credit Card UPI |

**Your existing providers already work! These are examples.**

---

## 📊 **Sheet 3: unrecognized_emails (Keep As-Is)**

This should already exist from your current setup. No changes needed.

---

## 📊 **Sheet 4: categories (CREATE NEW)**

### Create New Sheet:
1. Click "+" at bottom of Google Sheets to add new sheet
2. Rename it to: `categories`
3. Add headers in first row:

| category | subcategory | icon | notes |

### Import Data:
1. Download the `initial_categories.csv` file
2. In Google Sheets, go to "categories" sheet
3. File → Import → Upload → Select `initial_categories.csv`
4. Import location: "Replace data at selected cell"
5. Click "Import data"

**Result**: 45 pre-defined categories automatically added!

---

## 📊 **Sheet 5: merchant_mappings (CREATE NEW)**

### Create New Sheet:
1. Click "+" to add new sheet
2. Rename it to: `merchant_mappings`
3. Add headers in first row:

| merchant_key | category | subcategory | learned_at | count | notes |

### Leave Empty:
This will be populated automatically as CategoryMapper learns from your inputs.

---

## 📊 **Sheet 6: blocked_senders (CREATE NEW)**

### Create New Sheet:
1. Click "+" to add new sheet
2. Rename it to: `blocked_senders`
3. Add headers in first row:

| email_from | blocked_at | reason | notes |

### Leave Empty:
This will be populated automatically when you block senders via SmartOnboarding.

---

## ✅ **Verification Checklist**

After setup, verify you have these sheets:

```
□ transactions (updated with 9 new columns)
□ providers (existing, no changes needed)
□ unrecognized_emails (existing, no changes needed)
□ categories (newly created, 45 rows imported)
□ merchant_mappings (newly created, empty)
□ blocked_senders (newly created, empty)
```

---

## 🔧 **Quick Setup Commands**

### For Each New Sheet:

**1. Create "categories" sheet:**
```
- Click "+" at bottom
- Rename to "categories"
- Add headers: category, subcategory, icon, notes
- File → Import → initial_categories.csv
```

**2. Create "merchant_mappings" sheet:**
```
- Click "+"
- Rename to "merchant_mappings"
- Add headers: merchant_key, category, subcategory, learned_at, count, notes
- Leave empty
```

**3. Create "blocked_senders" sheet:**
```
- Click "+"
- Rename to "blocked_senders"
- Add headers: email_from, blocked_at, reason, notes
- Leave empty
```

**4. Update "transactions" sheet:**
```
- Click on column after "parsed"
- Right-click → Insert 9 columns to the right
- Add headers: provider_id, type, amount, currency, merchant, category, subcategory, raw_date, reference
```

---

## 📝 **Final Sheet Structure**

### transactions (Updated):
```
Old columns: id, threadId, snippet, payload, sizeEstimate, historyId, 
             internalDate, labels, To, Subject, From, parsed
New columns: provider_id, type, amount, currency, merchant, category, 
             subcategory, raw_date, reference
```

### providers (Existing):
```
Columns: provider_id, active, from_contains, subject_contains, body_contains,
         regex_amount, regex_merchant, regex_date, regex_reference, notes
```

### unrecognized_emails (Existing):
```
Columns: timestamp, gmail_message_id, email_from, email_subject, 
         snippet_or_body, provider_guess
```

### categories (New - 45 rows):
```
Columns: category, subcategory, icon, notes
```

### merchant_mappings (New - Empty):
```
Columns: merchant_key, category, subcategory, learned_at, count, notes
```

### blocked_senders (New - Empty):
```
Columns: email_from, blocked_at, reason, notes
```

---

## 🚀 **After Setup**

Once all sheets are configured:

1. ✅ PersonalExpenseTracker will populate the new transaction columns
2. ✅ SmartOnboarding will use categories and blocked_senders
3. ✅ CategoryMapper will use categories and merchant_mappings
4. ✅ Your Gmail metadata stays intact in old columns
5. ✅ New columns have clean, structured transaction data

## ⏱️ **Setup Time**

- **Update transactions sheet**: 2 minutes
- **Create categories + import CSV**: 3 minutes  
- **Create merchant_mappings**: 1 minute
- **Create blocked_senders**: 1 minute

**Total**: ~7 minutes to complete setup!

---

## 🆘 **Troubleshooting**

### If import fails:
1. Make sure CSV file is downloaded
2. Use File → Import (not copy-paste)
3. Select "Replace data at selected cell"
4. Check delimiter is comma

### If workflows error:
1. Verify sheet names match exactly (lowercase)
2. Check all required sheets exist
3. Ensure headers match exactly (case-sensitive)

Your Google Sheets setup is ready! 🎉
