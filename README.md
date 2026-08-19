# Invoice Intake & Bookkeeping Automation

An n8n workflow that watches an inbox, reads incoming invoice PDFs with Claude, files them by vendor in Google Drive, and opens the matching **unpaid** bill in QuickBooks.

**Built for:** Janey Health · **2026** · ~19s end-to-end per invoice

---

## The problem

Invoices arrived as PDF attachments in a shared inbox. For each one, someone had to:

1. Open the PDF and read off the vendor, amount, dates, and line items
2. Save it into the right vendor folder in Drive, under a consistent filename
3. Re-key the same numbers into QuickBooks as a bill on the correct expense account

Pure transcription — slow, easy to fat-finger, and exactly the kind of task that gets deferred until both the filing and the books have drifted.

## The approach

One workflow takes an invoice from unread email to filed document and open bill without anyone touching it.

The important design constraint was **where to stop**. The workflow creates the bill as *unpaid* and sends a review notification. It never schedules a payment. Automating data entry is safe; automating money leaving the account is not.

```
Gmail trigger
     │
     ▼
Explode PDF attachments ──► loop, one invoice at a time
     │
     ▼
Upload to Drive staging ──► Claude extracts fields + expense category
     │
     ▼
Find/create vendor folder ──► move + rename file consistently
     │
     ▼
Find/create QuickBooks vendor
     │
     ▼
Bill already exists? ──yes──► skip (no double-booking)
     │ no
     ▼
Create UNPAID bill on the mapped expense account
     │
     ▼
Notify for review ──► mark email as read
```

| Step | What happens |
|---|---|
| **Watch** | Gmail trigger fires on new mail in the invoices inbox |
| **Explode** | Each PDF attachment is split out and processed individually |
| **Extract** | Claude reads the PDF and returns structured invoice fields plus a category |
| **File** | Vendor folder found or created in Drive; file moved and renamed |
| **Vendor** | Matching QuickBooks vendor found, or auto-created if missing |
| **Dedupe** | Checks for an existing bill before writing |
| **Book** | Unpaid bill created on the account mapped from the category |
| **Close** | Review notification sent, email marked read |

Category → expense account routing lives in a single mapping node, so the accountant can change where something books without touching the workflow logic.

## Decisions worth explaining

**Stop at "unpaid bill", not at "paid".**
The workflow could have scheduled payment. It deliberately doesn't. The value is in eliminating transcription, and the risk profile of an AI-triggered payment is completely different from that of an AI-filed document. A human approves every dollar.

**Dedupe before writing, not after.**
Forwarded and resent invoice emails are normal. The workflow looks for an existing bill *before* creating one, so the same invoice arriving twice produces one bill rather than a reconciliation problem next month.

**A separate error-handling workflow.**
Failures route to their own handler rather than dying silently inside a loop. An invoice that can't be parsed is surfaced as a task for a person — so the automation degrades into the old manual process instead of losing the document.

**Vendors auto-create, accounts don't.**
A new vendor is low-risk and gets created automatically. A new *expense account* changes the shape of the books, so that stays a deliberate human action — there's a separate helper workflow for setting up the chart of accounts.

## Stack

n8n · Claude · QuickBooks Online API · Google Drive API · Gmail API

## Repository contents

- `workflows/` — exported workflow JSON, with credentials, realm/account IDs, and vendor data stripped

---

**[← More case studies](https://portfolio-eta-eight-46.vercel.app)**
