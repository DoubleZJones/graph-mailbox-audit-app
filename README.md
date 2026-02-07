# Mailbox-Scoped Microsoft Graph Audit Identity

This project implements a tightly scoped Microsoft Graph application designed to read only a specific set of mailboxes using app-only authentication. The goal was to create a predictable, least-privilege audit lane without granting broad Graph permissions or elevating any administrator’s personal account. Access is enforced through a mail-enabled security group and an Exchange Application Access Policy, ensuring the app can only see the mailboxes it is explicitly allowed to read.

---

## Overview

The core objectives of this project were:

- Build an app-only Graph identity using MSAL.PS.
- Restrict the app to two specific mailboxes using a mail-enabled security group.
- Enforce mailbox-level scoping with an Application Access Policy.
- Validate the configuration using Test-ApplicationAccessPolicy.
- Query messages, folders, and inbox rules without interactive login.
- Export audit data directly to JSON for offline analysis.

The result is a secure, repeatable audit identity aligned with Zero Trust principles and safe for multi-operator use.

---

## Phase 1: Creating the Mailbox-Scoped Application Identity

I began by creating a new Entra application configured for client-credentials authentication. This allowed the script to authenticate silently without any delegated permissions or user interaction.

To keep the app strictly read-only, I assigned only the permissions required for mailbox auditing:

- Mail.Read
- MailboxSettings.Read (required for inbox rules)

These permissions were granted at the application level and consented by an administrator.

---

## Phase 2: Scoping Access With a Mail-Enabled Security Group

To control which mailboxes the app could read, I created a mail-enabled security group and added the two target mailboxes. This group acts as the allow-list for the entire solution. Additional mailboxes can be added later without modifying the app or the policy.

This group became the foundation for the Application Access Policy.

---

## Phase 3: Enforcing Restrictions With an Application Access Policy

Using Exchange Online PowerShell, I created an Application Access Policy that binds:

- The app’s service principal
- The mail-enabled security group

This policy ensures the app can only access mailboxes that are members of the group. Everything else is hidden from the app entirely.

To verify the configuration, I used Test-ApplicationAccessPolicy, which confirmed:

- Allowed mailboxes returned AccessAllowed
- All other mailboxes returned AccessDenied

This validation step ensured the scoping was correct before any Graph calls were made.

---

## Phase 4: Querying Mailbox Data Using MSAL.PS

With the policy in place, I implemented a clean MSAL.PS token flow using a SecureString client secret. This allowed the script to authenticate silently and consistently.

I tested the following Graph endpoints:

- /messages for message retrieval
- /mailFolders for folder enumeration
- /mailFolders/inbox/messageRules for inbox rule auditing

Allowed mailboxes returned 200 OK, while restricted mailboxes returned 404 NotFound, confirming the policy was working as intended.

To avoid browser rendering issues with large JSON responses, all output was exported directly to disk.

---

## Impact

This project delivered a secure, reusable audit identity that stays within its defined boundaries. It eliminates the need for elevated admin roles or delegated mailbox permissions and provides a repeatable pattern for safe mailbox-level automation across Microsoft 365 environments.
