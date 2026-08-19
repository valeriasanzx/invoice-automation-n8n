# Exported workflows

## How to add them

1. In n8n, open each workflow → **⋯ menu → Download**
2. Save the JSON here:
   - `invoice-watcher.json` — Invoice Watcher: Extract & File to Drive
   - `invoice-watcher-error-handler.json` — Invoice Watcher - Error Handler
   - `qb-create-chart-of-accounts.json` — QB Helper: Create Chart of Accounts
3. **Sanitize before committing.** Remove or replace:
   - `credentials` blocks (IDs and names)
   - QuickBooks realm ID and any hardcoded account IDs
   - Google Drive folder IDs
   - Real vendor names in `vendorOverrides` / example data
   - The inbox address in the Gmail trigger
   - Any webhook URLs and workflow IDs

A quick check before pushing:

```bash
grep -riE 'realm|credential|@gmail|@janey|folderId|webhookId|apiKey' workflows/
```
