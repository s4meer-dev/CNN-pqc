# `src/classical/` — Classical Cryptography Baseline (Planned)

> **Status:** Blueprint only (Phase 3 deliverable).

## Planned Responsibility
Provide clean, uniform Python interfaces around industry-standard **OpenSSL / `cryptography`** primitives so classical methods act as an honest, fast baseline against PQC.

## Algorithms Planned Here
- **Key Sharing (Job 1):**
  - `RSA-2048` & `RSA-3072` (Key Transport / RSA-OAEP encapsulation baseline)
  - `ECC P-256` (`ECDH secp256r1`) and `ECC P-384` (`ECDH secp384r1`)
- **Digital Signatures (Job 2):**
  - `RSA-2048` & `RSA-3072` (RSA-PSS signing & verification)
  - `ECC P-256` (`ECDSA`) and `ECC P-384` (`ECDSA`)

## Common Interface Goal
Every wrapper will return exact serialized byte sizes (`public_key_bytes`, `private_key_bytes`, `ciphertext_or_sig_bytes`) and support isolated timing of `KeyGen`, `Exchange/Sign`, and `Decaps/Verify`.
