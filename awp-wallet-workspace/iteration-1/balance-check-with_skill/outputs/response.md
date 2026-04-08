# Balance Check Response: Base 链上的 USDC 余额查询

## User Request

> 帮我查一下我在 Base 链上有多少 USDC

---

## Step 1 — Ensure awp-wallet is installed

```bash
awp-wallet --version 2>/dev/null || {
  git clone https://github.com/awp-core/awp-wallet.git /tmp/awp-wallet &&
  cd /tmp/awp-wallet && bash install.sh
}
```

**Error handling:** If `awp-wallet: command not found` persists after install:
```bash
export PATH="$HOME/.local/bin:$PATH"
```

---

## Step 2 — Ensure wallet exists

```bash
awp-wallet receive 2>/dev/null || awp-wallet init
```

**Error handling:** If `No wallet found`, the `awp-wallet init` fallback will create one. In that case, inform the user:

> [WALLET] created new wallet: 0x... (address from init output)

---

## Step 3 — Unlock session

```bash
TOKEN=$(awp-wallet unlock --duration 3600 | jq -r '.sessionToken')
```

**Error handling:** If `Invalid or expired session`, re-run this command to get a fresh token.

---

## Step 4 — Query USDC balance on Base

```bash
awp-wallet balance --token $TOKEN --chain base --asset usdc
```

This returns JSON output with the USDC balance on the Base chain.

---

## Step 5 — Lock wallet when done

```bash
awp-wallet lock
```

---

## User-Facing Response

Assuming the command in Step 4 returns successfully, e.g.:

```json
{
  "chain": "base",
  "asset": "usdc",
  "balance": "1250.75",
  "usdValue": "$1,250.75"
}
```

The response to the user would be:

> [QUERY] 你在 Base 链上的 USDC 余额为: **1,250.75 USDC** (约 $1,250.75)

---

## Error Handling Summary

| Scenario | Detection | Action |
|----------|-----------|--------|
| `awp-wallet` not installed | `command not found` | Clone repo and run `install.sh`, then set PATH |
| No wallet exists | `No wallet found` | Run `awp-wallet init`, inform user of new address |
| Session token expired | `Invalid or expired session` | Re-run `awp-wallet unlock --duration 3600` |
| Insufficient balance / zero balance | Balance returns `"0"` | Inform user: `[QUERY] 你在 Base 链上的 USDC 余额为 0 USDC` |
| Network/RPC error | Command fails with network error | Retry once; if still failing, inform user of temporary network issue |

---

## Complete Command Sequence (copy-paste ready)

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

# 4. Check USDC balance on Base
awp-wallet balance --token $TOKEN --chain base --asset usdc

# 5. Lock wallet
awp-wallet lock
```
