# App Overview & Functionality

**Cardano-CLI Transaction Visualizer** is a pure-web (Next.js + React) application that lets non-technical users:

1. **Manage Keys Offline**  
   - Generate or import a wallet mnemonic in the browser  
   - Encrypt the private key with a passphrase and store it in IndexedDB  
   - Unlock it on demand, build & sign transactions entirely offline with Lucid

2. **Build & Sign Transactions**  
   - Query UTxOs (via Blockfrost or local API) and cache them  
   - Provide a simple “Send ADA” form—no raw flags  
   - Under the hood, Lucid builds the transaction, signs it with the user’s key, and exports a CBOR hex blob

3. **Event Logging & Audit Trail**  
   - Every key action (`key:created`, `key:unlocked`, `tx:signed`, `key:wiped`, `tx:submitted`) is recorded  
   - Logs are stored in a secure, append-only IndexedDB store for user review

4. **Offline-First UX**  
   - Users can download the signed CBOR and submit later (air-gapped if desired)  
   - When ready, they “Upload CBOR” to a Next.js API route  
   - The server (via **cardanocli-js**) submits it to a local `cardano-node` and returns the TxHash

---

# Storage Usage Estimation

| Store           | Content                                                    | Typical Size per Item | Example Count | Total Size  |
|-----------------|------------------------------------------------------------|-----------------------:|--------------:|-----------:|
| **KeyStore**    | AES-GCM encrypted private key + metadata                   | ~2 KB                  | 1             | ~2 KB       |
| **Key Logs**    | `{ ts, action, details, success }` entries                 | ~0.2 KB (200 B)        | 100           | ~20 KB      |
| **UTxO Cache**  | Cached UTxO objects (`txHash`, `index`, `lovelace`, datum) | ~0.5 KB (500 B)        | 100           | ~50 KB      |

**Estimated total** (1 wallet + 100 log entries + 100 UTxOs): **≈ 72 KB**

> **Notes & Scaling**  
> - **KeyStore** size is fixed per wallet (typically 1–3 KB).  
> - **Logs** grow linearly—200 bytes each; you can purge or archive old entries.  
> - **UTxO cache** depends on how many addresses you query; you might only keep the “active” ones (e.g. 20–50 UTxOs).  
> - Even with 1,000 UTxOs and 1,000 logs, you’re under 1 MB—well within modern browsers’ IndexedDB quotas.

This footprint is lightweight, ensures responsive UI, and keeps all user data completely under their control.
