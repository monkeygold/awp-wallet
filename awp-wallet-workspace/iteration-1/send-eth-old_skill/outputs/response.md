# Agent Response: "send 0.05 ETH to 0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18 on Arbitrum"

## Step 1: Ensure wallet is available and unlock session

### Commands

```bash
# 1. Check if awp-wallet is installed
awp-wallet --version

# 2. Ensure wallet exists (creates one if needed)
awp-wallet receive 2>/dev/null || awp-wallet init

# 3. Unlock the wallet and capture the session token
TOKEN=$(awp-wallet unlock --duration 3600 | jq -r '.sessionToken')
```

### Error handling -- if awp-wallet is not found

```bash
git clone https://github.com/awp-core/awp-wallet.git /tmp/awp-wallet
cd /tmp/awp-wallet && bash install.sh
```

Then retry the unlock sequence above.

### Error handling -- if No wallet found

```bash
awp-wallet init
```

Then retry unlock.

---

## Step 2: Estimate gas (optional, informational)

### Command

```bash
awp-wallet estimate --to 0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18 --amount 0.05 --chain arbitrum
```

### User-facing message

```
[QUERY] Estimating gas for 0.05 ETH transfer on Arbitrum...
```

---

## Step 3: Show confirmation prompt (Write Safety)

### User-facing message

```
[TX] about to send:
     to:      0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18
     amount:  0.05 ETH
     chain:   Arbitrum
     proceed? (y/n)
```

The agent waits for the user to confirm with "y" before proceeding. If the user replies "n" or anything other than "y", the transaction is aborted and the agent responds:

```
Transaction cancelled.
```

---

## Step 4: Execute the send (only after user confirms "y")

### Command

```bash
awp-wallet send --token $TOKEN --to 0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18 --amount 0.05 --chain arbitrum
```

Note: `$TOKEN` is the session token obtained from `awp-wallet unlock` in Step 1.

### User-facing message (on success)

```
[TX] Sent 0.05 ETH to 0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18 on Arbitrum.
     tx: https://arbiscan.io/tx/0x<transaction_hash>
```

The transaction hash and explorer link are extracted from the JSON output of the `awp-wallet send` command.

---

## Step 5: Lock the wallet

### Command

```bash
awp-wallet lock
```

---

## Error Handling Summary

| Scenario | Action |
|----------|--------|
| `awp-wallet: command not found` | Clone repo and run `install.sh`, then retry |
| `No wallet found` | Run `awp-wallet init`, then retry |
| `Invalid or expired session` | Re-run `awp-wallet unlock --duration 3600` to get a fresh token |
| `Insufficient balance` | Inform user: "Insufficient ETH balance on Arbitrum. Please fund your wallet or try --mode gasless if you have a PIMLICO_API_KEY configured." |
| `Daily limit exceeded` | Inform user: "Daily transaction limit exceeded. Please try again tomorrow." |
| Send command returns non-zero exit code | Parse the JSON error, display it to the user, and suggest corrective action based on the error table above |

## Complete Bash Script (sequential)

```bash
#!/usr/bin/env bash
set -euo pipefail

# 1. Ensure wallet exists
awp-wallet receive 2>/dev/null || awp-wallet init

# 2. Unlock and get session token
TOKEN=$(awp-wallet unlock --duration 3600 | jq -r '.sessionToken')

# 3. Estimate gas
echo "[QUERY] Estimating gas for 0.05 ETH transfer on Arbitrum..."
awp-wallet estimate --to 0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18 --amount 0.05 --chain arbitrum

# 4. Confirmation prompt (agent shows this to user and waits for "y")
# [TX] about to send:
#      to:      0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18
#      amount:  0.05 ETH
#      chain:   Arbitrum
#      proceed? (y/n)

# 5. Send (only after user confirms)
RESULT=$(awp-wallet send --token "$TOKEN" --to 0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18 --amount 0.05 --chain arbitrum)
TX_HASH=$(echo "$RESULT" | jq -r '.transactionHash // .hash')
echo "[TX] Sent 0.05 ETH to 0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18 on Arbitrum."
echo "     tx: https://arbiscan.io/tx/${TX_HASH}"

# 6. Lock wallet
awp-wallet lock
```
