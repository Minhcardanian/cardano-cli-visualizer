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
> - **KeyStore** size is fixed per wallet (1–3 KB).  
> - **Logs** grow linearly—200 bytes each; purge/archive old entries.  
> - **UTxO cache** depends on addresses queried; keep active ones only.  
> - Even with 1,000 UTxOs and 1,000 logs, you’re under 1 MB.

---

## Development Stack & Versioning

- **Node.js**: 18.x LTS  
- **Next.js**: ^13.4.0  
- **React**: ^18.2.0  
- **TypeScript**: ^5.1.0  
- **Lucid**: ^0.9.0  
- **@emurgo/cardano-serialization-lib-browser**: ^11.1.0  
- **cardanocli-js**: ^2.1.0  
- **Tailwind CSS**: ^3.4.0  
- **event-source-polyfill**: ^1.0.27  
- **cardano-node / cardano-cli**: v8.0.0+ (matching network era)

> Pin versions in `package.json` and use `npm ci` for reproducible installs.

---

## Master Architecture

```mermaid
flowchart TD
  subgraph OfflineSigning["Offline Signing (Browser)"]
    direction TB
    UI["User UI"]
    Unlock["Unlock Key\n(passphrase)"]
    KS["KeyStore\n(in-memory)"]
    LB["Lucid TX Builder"]
    SG["Lucid TX Signer"]
    Log["Event Logger"]
    Export["Export CBOR\n(File/QR)"]
    Clear["Clear Keys\nfrom Memory"]
    UI --> Unlock --> KS --> LB --> SG
    SG --> Log
    SG --> Export
    SG --> Clear
  end

  subgraph DBStores["IndexedDB Stores"]
    direction TB
    KSDB["KeyStore"]
    KLDB["Key Logs"]
    UTXODB["UTxO Cache"]
    KSDB -.-> KS
    KLDB -.-> Log
    UTXODB -.-> UI
  end

  Export --> API["Next.js API\n/api/submit"]

  subgraph OnlineSubmission["Online Submission (Server)"]
    direction TB
    CLIJS["cardanocli-js"]
    CLI["cardano-cli\ntransaction submit"]
    NODE["cardano-node"]
    API --> CLIJS --> CLI --> NODE --> CLIJS --> API
    API --> UI
  end
