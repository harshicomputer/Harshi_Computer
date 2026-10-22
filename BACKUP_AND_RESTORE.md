# Harshi Computer — Backup and Recovery Notes

## What the Full Site Backup contains

Use **Office / Salary Record → Backup / Export → एक क्लिक में पूरा Site Backup** while signed in as an authorized admin.

The protected Supabase Edge Function `full-site-backup`:
- checks the signed-in user against `public.app_admins`;
- exports all configured application tables (including encrypted Aadhaar payloads through the server-only service role);
- downloads every discovered object from the existing Supabase Storage buckets;
- creates a ZIP containing `database/full-site-database.json`, the document files under `storage/<bucket>/...`, and `manifest.json`;
- uploads the ZIP to the private `site-backups` bucket;
- records the database snapshot and manifest in `public.system_backups`;
- reopens the ZIP and verifies that the manifest, database JSON, and storage-file count are present before upload.

## Included application tables

`employees`, `salary_records`, `loan_accounts`, `loan_installments`, `loan_payments`, `loans`, `fdr_records`, `account_heads`, `account_transactions`, `bank_accounts`, `bank_transactions`, `employee_documents`, `employee_aadhaar_private`, `employee_contact_credentials`, `employee_bank_details`, `employee_personal_details`, `important_orders`, `business_customers`, `business_customer_documents`, `important_documents`, and `app_admins`.

`system_backups` itself is excluded from its own embedded snapshot to avoid recursive backups; the new backup entry is recorded separately in that table.

## Exclusions and limits

- Supabase Auth passwords, MFA secrets, API keys, and project secrets are not exported.
- The archive includes application data and Storage files, but it is not a full Supabase platform-level physical database dump.
- The source code and commit history are stored in the GitHub repository: https://github.com/harshicomputer/Harshi_Computer
- The archive integrity check is not the same as a disaster-recovery restore test.

## Required recovery test

Before claiming disaster-recovery readiness, restore the archive into a separate, isolated Supabase test project, recreate required schema/RLS/functions and storage buckets, load the database records and files, and test login/MFA, employees, salary, loan/EMI, FDR, accounting ledgers, and document downloads. Do not run a restore test against the production project. Creating a separate Supabase test project may incur charges and must be approved by the owner first.

## Security

The ZIP is stored in a private bucket and the app generates a short-lived signed download URL for an authenticated admin. Keep downloaded copies encrypted and in a restricted location. Never publish a backup archive or share its signed URL publicly.
