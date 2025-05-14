# User Requirements and Demo Layout

This document outlines the user requirements and the demo layout for our Cardano‑CLI web‑based shell experience. It serves as a guide for both developers and presenters.

---

## 1. User Requirements

* **Environment**:

  * Node.js (v16+) or Python (v3.9+) runtime
  * `cardano-cli` installed and configured (mainnet or testnet)
  * Network access (localhost or public URL via ngrok/Vercel)

* **CLI Workflow as Functions**:

  * Treat each `cardano-cli` invocation as a named function (e.g. `createKeys(wallet)`, `getBalance(address)`, `lockFunds(wallet, amount)`)
  * Use string‑templates with replaceable parameters for each command
  * Support a `--simulate` (dry‑run) flag for previews without on‑chain submission

* **Security & Validation**:

  * Input validation to prevent command injection
  * Sandbox execution (Docker/VM) to isolate demo environment
  * Config file for network flags and script addresses

* **Interactive Presentation Features**:

  * Typewriter animation for commands
  * Real‑time progress spinners with sub‑step labels (build, sign, submit)
  * Before/after state diffs (UTXO balances, pie charts)
  * Gamification elements (badges, success animations)
  * Inline tooltips for flags and parameters

* **Layout Options**:

  * **Split‑Page** (Left/Right):

    * **Left Pane**: Live terminal (xterm.js)
    * **Right Pane**: Visualization panel (charts, UTXO tables)
  * **Upper/Lower**:

    * **Top Pane**: Live terminal (xterm.js)
    * **Bottom Pane**: Visualization panel

* **Controls**:

  * Toolbar with playback controls: Prev | Next | Dry‑Run Toggle

---

## 2. Demo Layout

### 2.1. Pre‑requisites

1. Clone repository and install dependencies:

   ```bash
   git clone https://github.com/your-org/cardano-web-shell.git
   cd cardano-web-shell
   npm install    # or pip install -r requirements.txt
   ```
2. Ensure `cardano-node` and `cardano-cli` are running and synced.
3. (Optional) Start a tunnel:

   ```bash
   ngrok http 3000
   ```

### 2.2. File Structure

```plaintext
cardano-web-shell/
├─ cli-templates.js     # Command templates
├─ server.js            # WebSocket & PTY backend
├─ public/
│  └─ index.html        # Entry point
├─ src/
│  ├─ App.jsx           # Main React component
│  ├─ Terminal.jsx      # XTerm wrapper
│  └─ Visualization.jsx # Charts & tables
└─ config.json          # Network & script parameters
```

### 2.3. UI Components

| Component           | Description                                        |
| ------------------- | -------------------------------------------------- |
| **Terminal Pane**   | Live shell powered by xterm.js + node-pty          |
| **Viz Pane**        | Chart.js or D3.js for UTXO tables & balance graphs |
| **Toolbar**         | Step controls, dry‑run toggle, presentation‑mode   |
| **Parameter Panel** | Inline inputs/sliders to tweak function arguments  |

### 2.4. Upper/Lower Split View

```markdown
┌──────────────────────────────────────────────┐
│ Terminal (xterm.js)                         │
│ ┌───┐ ─▶ lockFunds("DemoWallet", 1_000_000) │
│ │ ▶ │ Execute Next Step                     │
│ └───┘                                      │
├──────────────────────────────────────────────┤
│ Visualization (Chord/UTXO)                  │
│ ┌──────────────┐                             │
│ │ Pie Chart    │                             │
│ └──────────────┘                             │
└──────────────────────────────────────────────┘
```

### 2.5. Split‑Page View

```markdown
┌────────────────────────────────┬───────────────────────────────┐
│ Terminal (xterm.js)           │ Visualization (Chord/UTXO)    │
│ ┌───┐ ─▶ lockFunds("Demo",1)   │ ┌──────────────┐               │
│ │ ▶ │ Execute Next Step        │ │ Pie Chart    │               │
│ └───┘                         │ └──────────────┘               │
└────────────────────────────────┴───────────────────────────────┘
```

### 2.6. Demo Script Steps

1. **Generate Keys**

   ```bash
   createKeys("DemoWallet")
   ```
2. **Build Address**

   ```bash
   buildAddress("DemoWallet")
   ```
3. **Get Balance**

   ```bash
   getBalance("addr_test1…")
   ```
4. **Lock Funds**

   ```bash
   lockFunds("DemoWallet", 1_000_000)
   ```
5. **Visual Confirmation**

   * Pie chart updates to reflect locked funds
6. **Unlock Funds**

   ```bash
   unlockFunds("DemoWallet", "<txIn>", "{}")
   ```
7. **State Diff**

   ```diff
   - Wallet: 5 ADA
   + Wallet: 4 ADA
   ```

### 2.7. Presentation Mode

* **Single‑pane Terminal**: Full‑screen with typewriter effect
* **Speaker Controls**: Hotkeys to toggle tooltips, pause playback, switch to recorded fallback

### 2.8. Fallback Strategy

* If live demo fails, play back a pre‑recorded `ttyrec` session
* Static sequence of commands & outputs with preserved timing

---

> *This layout ensures a seamless, interactive demo that clearly ties each `cardano-cli` function call to its visual outcome, keeping the audience engaged and informed.*
