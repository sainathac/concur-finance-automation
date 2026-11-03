# Concur Finance Automation

Month-end finance operations for SAP Concur — accrual coding, billing-detail reporting, and AP/GL cash extracts, automated end-to-end with Python + Playwright.

## Situation

Every month-end, three finance processes against the SAP Concur portal and Cognos reporting were done by hand: accrual coding (ATR), prior-month billing-detail retrieval, and the AP/GL cash extract cycle. It was slow, deadline-sensitive, and large assigned-claims downloads kept failing against Cognos row limits.

## Task

I was responsible for automating the full month-end cycle end to end — so the reports produce themselves, credentials never live in code, and a failed unattended run can be picked up without redoing the portal work by hand.

## Action

- I built a shared Playwright automation core that performs Okta + TOTP login, navigation, retries and configuration once, and reused it across all three jobs
- I moved every credential and path into environment configuration, so nothing sensitive exists in the repository
- I added run-state files and idempotency so each job resumes safely, and a run-ack lock so the one-shot cash extract can never fire twice
- I wired failure screenshots and structured logs into every job so triage does not require re-running the portal manually
- For the Cognos row limit, I split the assigned-claims download by year and merged the parts into one accrual workbook

## Flow (high level)

```
Scheduled trigger
      |
Secure portal login (Okta + TOTP)
      |
Run month-end job  (ATR . Billing detail . Cash extract)
      |
Validate + merge output
      |
Save run state  -- on failure: screenshot, resume next run
```

## Result

Three month-end processes that previously needed hands-on portal work at every close now run unattended; the cash extract triggers exactly once per cycle; reports that previously died on Cognos row limits complete via year-splitting; failed runs self-document with screenshots and resume cleanly on the next attempt.

---

*High-level overview only — this project is a confidential client engagement; implementation details are not published.*
