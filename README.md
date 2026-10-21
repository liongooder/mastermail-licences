# mastermail-licences

Public **revocation list** for **MasterMail-OS** («مراسلات ماستر»). This repository contains **only** one
signed, public file — `revoked.json` — a list of revoked **fingerprints**. It holds no customer names, no
licence codes and no machine IDs, and **no private key**. The MasterMail-OS app downloads `revoked.json` quietly
in the background, verifies its signature with the product's embedded public key, and locks any copy whose
licence-id or machine fingerprint appears in the list.

## File
- `revoked.json` — `{ "payload": base64url(UTF-8 JSON), "sig": base64url(Ed25519 signature over the payload) }`
  where the payload is `{"type":"mastermail-revocations","serial":<N>,"issued":"<ISO>","revoked":[<fingerprint>,…]}`.
- A fingerprint is `sha256("MMOS-REVOKE|" + kind + "|" + VALUE.strip().upper())` hex, first 32 chars, where
  `kind` is `licence` (a licence id) or `machine` (a Machine ID).
- A **higher `serial`** replaces a lower one. The app keeps the newest verified copy, so an older list can never
  undo a revocation. The current list is **empty** (serial 1) — nothing is revoked.

## Revoke / restore workflow (vendor only)
The signing key stays private (never in this repo). Use the vendor tool `keygen.py` in
`MasteOS-Private\MasterMail-OS_PRIVATE` (its **Revoke / Restore** window, or its CLI) to add or remove a
fingerprint; it bumps the serial, re-signs the list and pushes it here. In short:
1. Revoke a licence id or a Machine ID → the tool regenerates `revoked.json` (serial + 1, signed).
2. Commit and push `revoked.json` to this repo's `main` branch.
3. Each MasterMail-OS copy picks up the newer list at its next start (or hourly) while online and locks if listed.
4. **Restore** = publish a newer list without that fingerprint.

Only the signed public list lives here. Never commit the private key or any `signing_key.pem`.
