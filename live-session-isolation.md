---
name: Live session isolation
description: Why startup validation and session imports must not alter the active WhatsApp account's local authentication state.
---

Startup smoke checks share the live bot's filesystem even when they use a different HTTP port. They must validate modules and HTTP startup without importing a session, opening the live auth database for writes, or clearing the session directory. Normal restarts should keep an existing populated auth database rather than replacing its credentials from the session token.

WhatsApp's "Waiting for this message" placeholder can occur on the user's command or on the bot's reply in Message yourself. Determine which bubble shows it before diagnosing command parsing. On a bot reply, the problem may be linked-device encryption or resend recovery even when commands execute correctly.

**Why:** A screenshot showed readable commands followed by unreadable replies from the linked bot device. An online connection and passing command tests do not prove the phone can decrypt outgoing self-chat messages. The user confirmed a fresh ping reply became readable after resends could find saved messages across phone and LID identities. Later live logs showed resend misses even while the message store was below its per-chat limit, so a missing lookup alone is not proof of corrupted session keys.

**How to apply:** Check the message direction first. For unreadable self-chat replies, inspect retry lookups across phone and LID addresses and check whether outgoing storage has finished. Keep a reaction's destination consistent with the original message's chat address; keep group view-once reveals private. Preserve the auth database rather than resetting it based on a placeholder alone.

**Why:** A startup check once cleared the live auth database while the bot process was connecting; the HTTP health check still passed, but WhatsApp remained stuck in a retry loop. Preserving credentials and signal keys across process launches restored connection.

**How to apply:** If changing startup checks, session import, or reconnection, test that the probe leaves an already populated auth database untouched and that a normal workflow restart reconnects using retained credentials and keys. Do not treat a successful HTTP health response as proof that WhatsApp is connected.

For self-chat commands, use the authenticated account's phone and LID identities to decide whether an outgoing LID chat is actually the bot's own chat. Message metadata such as an entry-point hint is not identity evidence. Keep a device suffix from changing the JID's domain during normalization.

**Why:** An own-chat command can arrive under a LID without a phone-number alternate. Treating it as an unrelated chat sends the reply to that LID; matching the authenticated own LID allows the command reply to use the own phone identity. This is a bot-side routing correction, not proof that every WhatsApp decryption placeholder has the same cause.

**How to apply:** Normalize both the authenticated phone and LID, then compare exact user-and-domain identities before choosing the self-chat reply destination. Retain the original chat address in incoming and quoted keys for protocol actions such as reactions, deletes, and resends; do not reset authentication data to resolve an ambiguous LID.

Do not blindly restart a failed WhatsApp workflow after a logged-out response. The existing logout path removes the auth directory, and startup can recreate it from a stale saved session credential.

**Why:** A failed launch was followed by a 401 response and automatic auth-directory removal; another launch imported the saved credential again and received the same rejection. A fresh secure pairing was required before the bot could connect.

**How to apply:** Check failure status and reconnect logs before retrying. Do not expose session credentials in chat, logs, or shell commands. Request a replacement through the secure secrets flow; preserve any auth files that still exist until the user chooses a recovery path.

Simplifying the owner’s self-chat `.ping` to a single plain-text reply and suppressing both reactions did not change the phone’s “Waiting for this message” result after fresh pairing. The resend lookup succeeded for an own-LID primary-phone retry, but lookup success did not prove decryption.

**Why:** This controlled test rules out buttons and reactions as the sole cause and shows that the participant-specific retry branch is not used for this primary-phone case.

**How to apply:** Investigate the send-to-all retry’s device selection, address namespace, and encryption/session setup; verify any proposed fix by a new phone-side result rather than a sent-message log or successful resend lookup.

Accepting another bot's compressed session format is not consent to replace an already connected account. Keep the active authentication store until the user explicitly chooses an account switch; use fresh credentials through the secure secrets flow rather than a token pasted into chat.

**Why:** A format-valid external session can still lack compatible Signal keys or fail WhatsApp login, while switching accounts can discard the current working linked-device state.

**How to apply:** Test decoders with synthetic credentials and verify the current store survives startup. For a real account switch, explain the disruption first and require a fresh privately entered session plus an explicit choice to replace the active login. A successful parse is not proof that WhatsApp connected or the phone can decrypt messages.