# Encrypted offline HTML backup for authenticator codes (proposal)

A feature proposal for Bitwarden Authenticator: export your TOTP accounts as a **single, self-contained, password-encrypted HTML file** that shows your current 6-digit codes in any browser, offline, with no app and no server.

> **Status:** concept and proof of concept. The demo contains fictional accounts only. It has **not** been security-audited. Do not use it for real secrets.

## The problem

If you lose your phone, you lose your authenticator app. Recovery then becomes circular:

1. Bitwarden vault asks for a 2FA code.
2. The authenticator was on the lost phone.
3. The cloud backup of the authenticator lives in a Google account.
4. The Google password is in the Bitwarden vault, and Google asks for 2FA too.
5. Back to step 1.

Bitwarden Authenticator currently exports only **unencrypted JSON** with raw secrets. That file cannot generate codes by itself, and it exposes every secret in plain text.

## The proposal

Add an export option: **"Encrypted HTML backup"**.

The exported file:

- contains the authenticator data (JSON) **encrypted inside the file**, using a separate password chosen by the user;
- opens in any modern browser on any OS, offline, without installing anything;
- asks for the password, then shows **only account names and the current codes**, refreshing every 30 seconds;
- is **view-only**: it does not display the secret keys and cannot modify anything;
- makes **no network requests**; its source is plain, readable JavaScript.

The user can recover access to Bitwarden and every other account using only the file and the password.

## Security model

| Item | Proposal |
|---|---|
| Encryption | AES-256-GCM (authenticated; tampering is detected) |
| Key derivation | PBKDF2-SHA-256, 600,000+ iterations (Argon2id preferred in a production version) |
| Salt / IV | Random, unique per export |
| Code generation | RFC 6238 TOTP via the browser Web Crypto API |
| Network | None. Ideally enforced with a strict Content-Security-Policy |

**Honest limitations**

- The file can be copied and attacked offline with unlimited guesses. **Password strength is the real protection**; recommend a long passphrase.
- After unlocking, secrets exist in browser memory. "Not showing the secret" is a UI property, not a cryptographic one.
- A compromised device can capture the password as it is typed.
- The embedded JavaScript handles secrets, so it must be reviewed and audited.
- If the password is forgotten, there is no recovery. That is by design.

**Recommended export-screen warnings**

- Store the file somewhere reachable **without 2FA** (USB drive, memory card), not only in email or cloud accounts that require 2FA.
- Store the password **outside the vault** (memorized or on paper).
- Re-export after adding new accounts.

## Optional, not a replacement

This would be an optional recovery mechanism. It does not replace Bitwarden's existing two-step login or account recovery methods.

## Proof of concept

Open [`demo/index.html`](demo/index.html) in a browser. Demo password: `Exemplo-2FA-2026!`. It decrypts a bundled fictional backup locally and shows live TOTP codes.

## Where to discuss

Bitwarden collects feature requests on the community forum: https://community.bitwarden.com (Feature Requests category).

## License

MIT for the demo code (see `LICENSE`).
