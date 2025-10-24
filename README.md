# Mnemonic Wallet Checker (Tauri v2 + React + ethers)

A lightweight desktop app to derive Ethereum addresses from a mnemonic and display balances from multiple JSON‑RPC providers. Built with Tauri v2, Vite, React, and ethers v6.

## Features
- Derives addresses using BIP44 path `m/44'/60'/0'/0/i`
- Queries balances from multiple providers:
  - Ethereum: `https://eth.llamarpc.com`
  - Codex: `https://node-mainnet.codexnetwork.org` (expects EVM‑compatible JSON‑RPC)
- Cross‑platform packaging via Tauri: macOS `.app` and `.dmg`; Windows via CI
- Simple UI with an address table and two balance columns

## Screenshots
- Enter mnemonic → click `Show Addresses` → table lists addresses with ETH and Codex balances.

## Project Structure
```
mnemonic-wallet-checker/
├── src/                  # React frontend (Vite)
│   ├── App.jsx           # UI and balance logic
│   └── index.html        # Vite template used by dev server
├── index.html            # Root index for Vite dev URL
├── src-tauri/            # Tauri v2 Rust app
│   ├── src/main.rs       # Tauri main
│   ├── tauri.conf.json   # Tauri v2 config
│   ├── build.rs          # tauri_build integration
│   └── icons/            # Generated app icons (icns, ico, png, etc.)
└── .github/workflows/tauri.yml  # CI for macOS and Windows
```

## Prerequisites
- Node.js `>= 18` (tested with `20`)
- Rust toolchain (stable), Cargo
- Tauri CLI (installed as devDependency, used via `npm scripts`)
- macOS build only: Xcode command line tools (`xcode-select --install`)
- Windows build (locally): Visual Studio Build Tools; or use GitHub Actions in CI

## Getting Started
1. Install dependencies:
   ```bash
   npm ci
   ```
2. Run in development (starts Vite and Tauri):
   ```bash
   npm run dev
   ```
3. Open the Tauri window and enter your 12/24‑word mnemonic, then click `Show Addresses`.

## Build & Packaging
- Production build (macOS local):
  ```bash
  npm run build
  ```
  Artifacts will be under:
  - `src-tauri/target/release/bundle/macos/<Product>.app`
  - `src-tauri/target/release/bundle/dmg/<Product>_<version>_<arch>.dmg`

- Windows packaging:
  - Use the included GitHub Actions workflow (`.github/workflows/tauri.yml`).
  - It builds on `macos-latest` and `windows-latest` and uploads bundle artifacts.

### Code Signing (Optional for distribution)
- macOS: unsigned builds can be opened via right‑click → Open.
- For signed releases, configure Developer ID and add secrets (e.g., `TAURI_PRIVATE_KEY`, `TAURI_KEY_PASSWORD`) to your repo and follow Tauri docs.
- Windows signing requires a code‑signing cert; optional in CI.

## Configuration (Tauri v2)
Key settings in `src-tauri/tauri.conf.json`:
- Root: `identifier`, `productName`, `version`.
- `build.devUrl`: `http://localhost:5173`
- `build.frontendDist`: `../dist` (Vite output)
- `app.windows`: window title/size
- `bundle`: `active: true`, `targets: "all"`, and icons

If you see a warning about the bundle identifier ending with `.app`, change `identifier` to something like `com.walletviewer`.

## Customizing Providers
Update providers in `src/App.jsx`:
```js
const ethProvider = new ethers.JsonRpcProvider("https://eth.llamarpc.com");
const codexProvider = new ethers.JsonRpcProvider("https://node-mainnet.codexnetwork.org");
```
- If Codex is not EVM‑compatible, `getBalance` may fail; the UI shows `-` on failure.
- You can add more providers using the same `Promise.allSettled` pattern.

## Security Notes
- Never share your mnemonic. This app derives addresses locally but still makes RPC calls; use trusted providers.
- For production use, consider running in offline mode or with self‑hosted RPC.

## Troubleshooting
- White screen in Tauri window:
  - Ensure root `index.html` exists and `build.devUrl` is correct.
  - Open DevTools (Cmd+Opt+I on macOS) to check console errors.
- HD derivation error (non‑zero depth):
  - Initialize at root path `m`: `ethers.HDNodeWallet.fromPhrase(mnemonic, undefined, "m")`.
- Missing icons error in `tauri::generate_context!()`:
  - Generate icons: `npx tauri icon app-icon.svg` (puts assets under `src-tauri/icons`).
- macOS identifier warning:
  - Set `identifier` to not end with `.app`.

## CI: GitHub Actions
- Workflow: `.github/workflows/tauri.yml`
- Triggers: push to `main` or manual dispatch
- Outputs: bundle artifacts uploaded per platform

## Create & Push Repo (optional)
```bash
# Authenticate GitHub CLI
gh auth login

# Create repo from current directory and push
gh repo create --source . --public --remote origin --push
```
Or set remote and push with plain Git:
```bash
git remote add origin https://github.com/<your-username>/mnemonic-wallet-checker.git
git push -u origin main
```

## License
This project is intended for personal and educational use. Add a license file if you plan to distribute.