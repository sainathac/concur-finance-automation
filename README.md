# Concur Finance Automation

Month-end finance operations for SAP Concur — accrual coding, billing-detail reporting, and AP/GL cash extracts, automated end-to-end with Python + Playwright.

## What it does

Replaces three manual month-end processes against the SAP Concur portal and Cognos reporting: accrual coding (ATR), prior-month billing-detail retrieval, and the AP/GL cash extract cycle.

## Flow (high level)

```
Scheduled trigger
      ↓
Secure portal login (Okta + TOTP)
      ↓
Run month-end job  (ATR · Billing detail · Cash extract)
      ↓
Validate + merge output
      ↓
Save run state  ── on failure → screenshot, resume next run
```

## Logic

- Browser automation (Playwright) drives the portal; a shared core handles Okta + TOTP auth, retries and configuration
- Credentials come from environment configuration only — never from code
- Run-state files make every job idempotent and resumable; failed runs leave screenshots for triage

## Problems solved

- **Cognos row limits** cap the assigned-claims report → download split by year, merged into a single accrual workbook
- **One-shot cash extract** risked firing twice in unattended runs → run-ack lock guarantees exactly one trigger
- **Inconsistent portal sessions** → shared auth module with TOTP and retry logic instead of per-job logins

---

*High-level overview only — this project is a confidential client engagement; implementation details are not published.*
