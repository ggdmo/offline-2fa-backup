# Encrypted offline HTML backup for authenticator codes

An open idea for any authenticator app: export your TOTP accounts as a **single, self-contained, password-encrypted HTML file** that shows your current 6-digit codes in any browser, offline, with no app and no server.

**[Open the live demo](https://ggdmo.github.io/offline-2fa-backup/demo/)** (demo password: `Exemplo-2FA-2026!`)

> **Status:** concept and proof of concept. The demo contains fictional accounts only. It has **not** been security-audited. Never enter real secrets or real passwords into it.

Idea and proof of concept by [ggdmo](https://github.com/ggdmo).

## The problem

If you lose your phone, you lose your authenticator app. Recovery then becomes circular:

1. Your password manager asks for a 2FA code.
2. The authenticator was on the lost phone.
3. The cloud backup of the authenticator lives in an account (Google, Apple, Microsoft...).
4. That account's password is in the password manager, and the account asks for 2FA too.
5. Back to step 1.

Most authenticators export either nothing, or **unencrypted JSON** with raw secrets. That file cannot generate codes by itself, and it exposes every secret in plain text.

## The idea

An export option: **"Encrypted HTML backup"**.

The exported file:

- contains the authenticator data **encrypted inside the file**, using a separate password chosen by the user;
- opens in any modern browser on any OS, offline, without installing anything;
- asks for the password, then shows **only account names and the current codes**, refreshing every 30 seconds;
- is **view-only**: it does not display the secret keys and cannot modify anything;
- makes **no network requests**; its source is plain, readable JavaScript.

With the file and the password, the user can get back into every account.

## Security model

| Item | Proposal |
|---|---|
| Encryption | AES-256-GCM (authenticated; tampering is detected) |
| Key derivation | PBKDF2-SHA-256, 600,000+ iterations (Argon2id preferred in a production version) |
| Salt / IV | Random, unique per export |
| Code generation | RFC 6238 TOTP via the browser Web Crypto API |
| Network | None. Ideally enforced with a strict Content-Security-Policy |

**Honest limitations**

- The file can be copied and attacked offline with unlimited guesses. **Password strength is the real protection**; use a long passphrase.
- After unlocking, secrets exist in browser memory. "Not showing the secret" is a UI property, not a cryptographic one.
- A compromised device can capture the password as it is typed.
- The embedded JavaScript handles secrets, so it must be reviewed and audited.
- If the password is forgotten, there is no recovery. That is by design.

**Recommended warnings for any app that offers this export**

- Store the file somewhere reachable **without 2FA** (USB drive, memory card), not only in email or cloud accounts that require 2FA.
- Store the password **outside the password manager** (memorized or on paper).
- Re-export after adding new accounts.

## Optional, not a replacement

This is meant as an extra recovery method. It does not replace recovery codes or the recovery options each service already offers.

## Proof of concept

**[Open the live demo](https://ggdmo.github.io/offline-2fa-backup/demo/)**

Demo password: `Exemplo-2FA-2026!`

It decrypts a bundled fictional backup locally in your browser and shows live TOTP codes. No network requests. The source code is in the `demo` folder of this repository.

## Roadmap

- A generator page: paste or import an exported authenticator file, choose a password, download the encrypted HTML backup.
- Support for common export formats from popular authenticator apps.
- Independent security review.

Ideas and feedback are welcome in the Issues tab.

## License

MIT (see `LICENSE`). Copyright (c) 2026 ggdmo.
