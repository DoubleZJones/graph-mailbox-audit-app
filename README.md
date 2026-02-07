Mailbox‑Scoped Microsoft Graph Audit Identity
Security & Compliance Engineering Entry
Overview
This entry documents the work to build a tightly scoped, app‑only Microsoft Graph identity that can read from only a small set of mailboxes. The goal was to create a secure, predictable audit lane without giving anyone elevated roles or broad Graph permissions. Access is locked down using a mail‑enabled security group, an Exchange Application Access Policy, and a clean MSAL.PS authentication flow.

This pattern gives us a safe, repeatable way to automate mailbox audits while staying aligned with Zero Trust and least‑privilege principles.

1. Mailbox‑Scoped Graph Application Identity
Summary
I created an Entra application that authenticates using client credentials and is restricted to only the mailboxes we explicitly allow. Access is controlled through a mail‑enabled security group and an Application Access Policy created in PowerShell, which together ensure the app can’t read or even see any other mailbox in the tenant.

Key Actions
Registered a new Entra application for app‑only Graph access.

Assigned Mail.Read and MailboxSettings.Read (Application) permissions.

Created a mail‑enabled security group and added the two target mailboxes.

Additional mailboxes can be added to this group later as needed.

Created the Application Access Policy using PowerShell (New-ApplicationAccessPolicy) to bind:

The app’s service principal

The mail‑enabled security group as the allowed scope

Validated the policy using Test-ApplicationAccessPolicy to confirm the app could only access the intended mailboxes.

Allowed mailboxes returned AccessAllowed

Everything else returned AccessDenied

Verified real Graph behavior:

Allowed mailboxes return 200 OK

Restricted mailboxes return 404 NotFound

Added support for reading inbox rules via /mailFolders/inbox/messageRules.

Documented how to interpret 401, 403, and 404 errors for future troubleshooting.

Impact
Delivered a safe, reusable audit identity that stays within its boundaries.

Removed the need for elevated admin roles or delegated mailbox permissions.

Created a repeatable pattern for secure mailbox‑level automation going forward.

2. MSAL.PS Token Flow for Secure Automation
Summary
Set up a clean MSAL.PS‑based authentication flow that uses a SecureString client secret. This allows the script to run non‑interactively—no browser windows, no prompts—and makes it easy for multiple admins to use the tool safely.

Key Actions
Installed and configured MSAL.PS.

Converted the client secret into a SecureString for safer handling.

Built a reusable function that acquires tokens with Get-MsalToken.

Standardized the Bearer token header for all Graph calls.

Ensured the script never triggers interactive login or browser pop‑ups.

Impact
Provided a secure, automation‑friendly authentication method.

Enabled consistent use across multiple operators.

Removed dependency on user accounts or delegated permissions entirely.

3. Graph Mailbox Audit Endpoints
Summary
Documented and validated the Graph endpoints needed for mailbox auditing, including messages, folders, and inbox rules. Confirmed that all endpoints behave correctly within the app’s restricted scope.

Key Actions
Queried /messages for message retrieval.

Queried /mailFolders for folder enumeration.

Queried /mailFolders/inbox/messageRules for inbox rule auditing.

Exported JSON results directly to disk to avoid browser rendering issues.

Verified consistent 200/404 behavior across allowed and restricted mailboxes.

Impact
Established a clear, predictable audit workflow using Graph.

Ensured consistent behavior across all mailbox‑scoped operations.

Created a foundation for future enhancements (delta queries, attachments, etc.).

About
Real‑world identity, governance, and automation engineering focused on secure, least‑privilege patterns that scale across Microsoft 365 environments.
