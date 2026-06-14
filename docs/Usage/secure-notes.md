---
order: 72
---

# Secure notes

Secure notes let you store free-form, sensitive text alongside your 2FA accounts: recovery codes, backup phrases, passwords, or any other secret you want to keep with the rest of your vault.

## End-to-end encryption

Secure notes are **end-to-end encrypted** when vault encryption is enabled. The title and content are encrypted by your browser before they are sent to the server, and the server only ever stores opaque ciphertext. This means:

- The server cannot read your note titles or content.
- Notes can only be decrypted on a device where the vault is unlocked.
- If you lose your master password, your notes are unrecoverable.

When encryption is not enabled, notes are stored as-is. Use encryption to keep notes confidential.

## Creating a note

1. Navigate to **Secure Notes** (the notes icon in the navigation bar).
2. Click **New note**.
3. Enter a **title** and the note **content**.
4. Optionally choose a **content type**: _plain_ or _markdown_.
5. Click **Save**.

The note is encrypted client-side and sent to the server as ciphertext.

## Markdown notes

When the content type is set to _markdown_, the content is rendered as formatted Markdown when you view the note. Use Markdown for structured content such as:

- Recovery code lists
- Step-by-step procedures
- Links and code blocks

The stored bytes are still ciphertext; Markdown rendering happens in your browser after decryption.

## Pinning a note

Pin frequently used notes to keep them at the top of the list. Click the pin icon on a note card, or use the pin action from the note detail view. Pinned notes are sorted above unpinned ones.

## Editing and deleting

Open a note to edit its title or content. Changes are re-encrypted client-side on save. Deleting a note is permanent and cannot be undone — deleted notes are removed from the server immediately.

!!!warning Export
Secure notes are **not** included in the standard `.vault` account backup. Keep a separate, encrypted copy of any note content you cannot afford to lose.
!!!
