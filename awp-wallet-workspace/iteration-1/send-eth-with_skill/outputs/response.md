# AWP Wallet Skill Response: Send 0.05 ETH on Arbitrum

**User request:** "send 0.05 ETH to 0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18 on Arbitrum"

---

## Step 1 — Ensure Installed

```bash
awp-wallet --version 2>/dev/null || {
  git clone https://github.com/awp-core/awp-wallet.git /tmp/awp-wallet &&
  cd /tmp/awp-wallet && bash install.sh
}
```

If `command not found` persists:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

**User-facing message (if install was needed):**
> Installed awp-wallet. Proceeding...

---

## Step 2 — Ensure Wallet Exists

```bash
awp-wallet receive 2>/dev/null || awp-wallet init
```

**User-facing message (if new wallet created):**
> `[WALLET]` created new wallet: 0x... (the address returned by `init`)

**User-facing message (if wallet already exists):**
> (no output, proceed silently)

---

## Step 3 — Unlock Session

```bash
TOKEN=$(awp-wallet unlock --duration 3600 | jq -r '.sessionToken')
```

**Error handling:** If output contains `Invalid or expired session` or `TOKEN` is empty/null, retry once:

```bash
TOKEN=$(awp-wallet unlock --duration 3600 | jq -r '.sessionToken')
```

If it fails again, tell the user:
> Could not unlock the wallet session. Please check that the wallet is properly initialized and try again.

---

## Step 4 — Confirm with User Before Sending

**User-facing confirmation prompt:**

```
[TX] about to send:
     to:      0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18
     amount:  0.05 ETH
     chain:   Arbitrum
     proceed? (y/n)
```

**Wait for user confirmation.** Only proceed if user responds "y" or "yes".

If user declines:
> Transaction cancelled.

---

## Step 5 — Execute the Send (after user confirms "y")

```bash
awp-wallet send \
  --token $TOKEN \
  --to 0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18 \
  --amount 0.05 \
  --chain arbitrum
```

**User-facing message on success (parse JSON output for tx hash):**

> `[TX]` sent 0.05 ETH to 0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18 on Arbitrum (hash: 0x...)
>
> Explorer: https://arbiscan.io/tx/0x...

---

## Step 6 — Lock Session

```bash
awp-wallet lock
```

---

## Error Handling Summary

| Scenario | Detection | Action |
|----------|-----------|--------|
| `awp-wallet` not found | `command not found` in Step 1 | Clone repo and run `install.sh`; export PATH |
| No wallet exists | `No wallet found` in Step 2 | Run `awp-wallet init`, report new address to user |
| Session token invalid/expired | `Invalid or expired session` in Step 3 or Step 5 | Re-run `awp-wallet unlock --duration 3600` |
| Insufficient balance | `Insufficient balance` in Step 5 output | Tell user: "Insufficient ETH balance on Arbitrum. Please fund your wallet first." If no native gas, suggest `--mode gasless` if `PIMLICO_API_KEY` is set |
| Daily limit exceeded | `Daily limit exceeded` in Step 5 output | Tell user: "Daily sending limit exceeded. Please try again in 24 hours." |
| Network/RPC error | Connection or timeout errors | Retry the send command once. If it fails again, report the error to the user |

---

## Complete Bash Command Sequence (in order)

```bash
# 1. Ensure installed
awp-wallet --version 2>/dev/null || {
  git clone https://github.com/awp-core/awp-wallet.git /tmp/awp-wallet &&
  cd /tmp/awp-wallet && bash install.sh
}

# 2. Ensure wallet exists
awp-wallet receive 2>/dev/null || awp-wallet init

# 3. Unlock session
TOKEN=$(awp-wallet unlock --duration 3600 | jq -r '.sessionToken')

# 4. (Confirmation prompt shown to user — awaiting "y")

# 5. Send transaction
awp-wallet send \
  --token $TOKEN \
  --to 0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18 \
  --amount 0.05 \
  --chain arbitrum

# 6. Lock session
awp-wallet lock
```
