# Google Sheets Structure Reference

## 📊 Complete Sheet Structure for PersonalExpenseTracker

Your Google Sheets document should have **6 sheets** total:

---

## 1. **transactions** (Main Data)

### Headers:
| timestamp | gmail_message_id | provider_id | type | amount | currency | merchant | category | subcategory | email_from | email_subject | raw_date | raw_body | reference |

### Description:
- **timestamp**: When transaction was captured (auto-generated)
- **gmail_message_id**: Unique Gmail message ID (deduplication key)
- **provider_id**: Which provider parsed this (e.g., hdfc_credit, paytm_upi)
- **type**: DEBIT or CREDIT
- **amount**: Transaction amount (number)
- **currency**: INR, USD, etc.
- **merchant**: Merchant/store name
- **category**: Main category (e.g., Food, Transport)
- **subcategory**: Specific type (e.g., Restaurants, Fuel)
- **email_from**: Source email address
- **email_subject**: Email subject line
- **raw_date**: Date extracted from email
- **raw_body**: Email snippet/body
- **reference**: Transaction reference number

### Example Row:
| 2025-11-26 10:30 | abc123xyz | hdfc_credit | DEBIT | 450.00 | INR | Swiggy | Food | Restaurants | alerts@hdfcbank.com | Transaction Alert | 26-Nov-2025 | Purchase at Swiggy... | REF12345 |

---

## 2. **providers** (Parsing Rules)

### Headers:
| provider_id | active | from_contains | subject_contains | body_contains | regex_amount | regex_merchant | regex_date | regex_reference | notes |

### Description:
- **provider_id**: Unique identifier (e.g., hdfc_credit, amazon_in)
- **active**: TRUE or FALSE (enables/disables provider)
- **from_contains**: Email address filter (e.g., alerts@hdfcbank.com)
- **subject_contains**: Subject keyword filter (optional)
- **body_contains**: Body keyword filter (optional)
- **regex_amount**: Regex to extract amount (e.g., `Rs\.?\s*([0-9,]+\.[0-9]{2})`)
- **regex_merchant**: Regex to extract merchant name
- **regex_date**: Regex to extract transaction date
- **regex_reference**: Regex to extract reference number
- **notes**: Human-readable description, examples

### Example Row:
| hdfc_credit | TRUE | alerts@hdfcbank.com | transaction | purchase | Rs\\.?\\s*([0-9,]+\\.[0-9]{2}) | at\\s+(.+?)\\s+on | on\\s+(\\d{2}-\\w{3}-\\d{4}) | Ref:\\s*(\\w+) | HDFC Credit Card alerts |

### Auto-Generated Provider Row (from SmartOnboarding):
| amazon_in | TRUE | pay-notifications@amazon.in | | | (?:Rs\\.?|₹)\\s*([0-9,]+) | (Amazon) | | | Auto-generated. Amount: 1299, Merchant: Amazon |

---

## 3. **unrecognized_emails** (Review Queue)

### Headers:
| timestamp | gmail_message_id | email_from | email_subject | snippet_or_body | provider_guess |

### Description:
- **timestamp**: When email was captured
- **gmail_message_id**: Unique Gmail message ID
- **email_from**: Sender email address
- **email_subject**: Email subject
- **snippet_or_body**: Email content preview
- **provider_guess**: Best-guess provider if partial match

### Example Row:
| 2025-11-26 10:30 | xyz789abc | newsletter@flipkart.com | Weekend Sale | Don't miss... | (empty) |

### Purpose:
- Collects emails that couldn't be parsed
- SmartOnboarding reviews these periodically
- Either becomes a provider or gets blocked

---

## 4. **blocked_senders** (Permanently Ignored) 🆕

### Headers:
| email_from | blocked_at | reason | notes |

### Description:
- **email_from**: Email address to block
- **blocked_at**: Timestamp when blocked
- **reason**: Why blocked (e.g., "User confirmed not transaction", "Marketing email")
- **notes**: Optional additional context

### Example Rows:
| newsletter@flipkart.com | 2025-11-26 10:45:00 | User confirmed not transaction | Weekly marketing emails |
| promo@zomato.com | 2025-11-26 11:30:00 | User confirmed not transaction | Promotional offers |
| social@linkedin.com | 2025-11-26 12:00:00 | User confirmed not transaction | Social notifications |

### Purpose:
- Permanently ignore non-transaction emails
- SmartOnboarding filters these out automatically
- Reduces noise in unrecognized queue
- Can be manually reviewed/edited if needed

---

## 5. **categories** (Category Definitions)

### Headers:
| category | subcategory | icon | notes |

### Description:
- **category**: Main category name (e.g., Food, Transport)
- **subcategory**: Specific type under category (e.g., Restaurants, Fuel)
- **icon**: Emoji for visual identification
- **notes**: Description, examples

### Example Rows:
| Food | Groceries | 🛒 | Supermarkets, vegetables, daily essentials |
| Food | Restaurants | 🍽️ | Dining out, food delivery apps |
| Transport | Fuel | ⛽ | Petrol, diesel, CNG |
| Shopping | Electronics | 📱 | Gadgets, phones, laptops |

### Source:
Import from `initial_categories.csv` (45 pre-defined categories provided)

### Purpose:
- Defines available categories for categorization
- Used by CategoryMapper to show options
- Can add more anytime

---

## 6. **merchant_mappings** (Learned Associations) 🆕

### Headers:
| merchant_key | category | subcategory | learned_at | count | notes |

### Description:
- **merchant_key**: Merchant name (lowercase, normalized)
- **category**: Assigned category
- **subcategory**: Assigned subcategory
- **learned_at**: When mapping was created
- **count**: Number of times this merchant appears (optional)
- **notes**: Additional context (optional)

### Example Rows:
| swiggy | Food | Restaurants | 2025-11-26 10:00:00 | 15 | Food delivery app |
| shell | Transport | Fuel | 2025-11-26 10:15:00 | 8 | Petrol station |
| amazon | Shopping | Electronics | 2025-11-26 10:30:00 | 23 | Online shopping |
| starbucks | Food | Coffee & Snacks | 2025-11-26 11:00:00 | 12 | Coffee chain |

### Purpose:
- Auto-categorization of known merchants
- CategoryMapper checks this before asking user
- Learns from every categorization you make
- Grows automatically over time

---

## 📈 **Sheet Growth Over Time**

### Week 1:
- **providers**: 5-10 rows (manual + auto-generated)
- **blocked_senders**: 3-5 rows (newsletters, spam)
- **merchant_mappings**: 20-30 rows (as you categorize)
- **transactions**: Growing daily

### Month 1:
- **providers**: 15-20 rows (most sources covered)
- **blocked_senders**: 10-15 rows (most spam blocked)
- **merchant_mappings**: 100+ rows (auto-categorization improving)
- **transactions**: 200-500 rows

### Month 3+:
- **providers**: 20-25 rows (complete coverage)
- **blocked_senders**: 20-30 rows (fully filtered)
- **merchant_mappings**: 200+ rows (90%+ auto-categorization)
- **transactions**: 1000+ rows (full history)

---

## 🔧 **Maintenance Tasks**

### Weekly:
- Review unrecognized_emails (should be decreasing)
- Check if any blocked_senders need unblocking
- Verify new providers are working correctly

### Monthly:
- Archive old transactions if >5000 rows
- Review merchant_mappings for duplicates/typos
- Optimize provider regex patterns if needed

### As Needed:
- Add new categories to categories sheet
- Merge similar merchant names in merchant_mappings
- Refine auto-generated provider regex patterns

---

## 📥 **Quick Setup Checklist**

```
□ Create "blocked_senders" sheet
  Headers: email_from, blocked_at, reason, notes

□ Create "merchant_mappings" sheet
  Headers: merchant_key, category, subcategory, learned_at, count, notes

□ Update "transactions" sheet
  Add columns: category, subcategory

□ Create "categories" sheet
  Import initial_categories.csv (45 categories)

□ Verify "providers" sheet exists
  (Should already exist from your current setup)

□ Verify "unrecognized_emails" sheet exists
  (Should already exist from your current setup)
```

---

## 🎯 **Data Flow Summary**

```
Gmail → ExpenseTracker
           ├─→ Known Provider → transactions (category empty)
           └─→ Unknown → unrecognized_emails
                          ↓
                    SmartOnboarding (every 3hr)
                          ├─→ Not Transaction → blocked_senders
                          └─→ Is Transaction → New provider created
                                              → Reprocesses ALL same-sender emails
                                              → Moves to transactions
                          ↓
                    CategoryMapper (every 2hr)
                          ├─→ Known Merchant → Auto-categorize
                          └─→ Unknown → Ask user via Telegram
                                      → Save to merchant_mappings
```

All sheets work together to create a fully automated, self-learning expense tracking system! 🚀
