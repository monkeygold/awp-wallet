# Balance Check Response: Base 链上的 USDC 余额

## User Request

> 帮我查一下我在 Base 链上有多少 USDC

---

## Step 1: Check if awp-wallet is installed

```bash
awp-wallet --version
```

**If command not found** (error handling):

```bash
git clone https://github.com/awp-core/awp-wallet.git /tmp/awp-wallet
cd /tmp/awp-wallet && bash install.sh
```

> User-facing message (if install needed):
>
> `[WALLET] awp-wallet 未安装，正在为你安装...`

---

## Step 2: Ensure wallet exists

```bash
awp-wallet receive 2>/dev/null || awp-wallet init
```

**If a new wallet is created**, user-facing message:

> `[WALLET] creating new wallet...`
> `[WALLET] ready: 0x<address>`

---

## Step 3: Unlock the wallet session

```bash
TOKEN=$(awp-wallet unlock --duration 3600 | jq -r '.sessionToken')
```

**Error handling**: If unlock fails with "No wallet found":

```bash
awp-wallet init
TOKEN=$(awp-wallet unlock --duration 3600 | jq -r '.sessionToken')
```

**Error handling**: If unlock fails with "Invalid or expired session":

```bash
TOKEN=$(awp-wallet unlock --duration 3600 | jq -r '.sessionToken')
```

---

## Step 4: Query USDC balance on Base

```bash
awp-wallet balance --token $TOKEN --chain base --asset usdc
```

Expected JSON output (example):

```json
{
  "chain": "base",
  "asset": "usdc",
  "balance": "142.50",
  "address": "0xYourAddress..."
}
```

---

## Step 5: Lock the wallet

```bash
awp-wallet lock
```

---

## Step 6: User-facing response

Based on the returned JSON, the agent would reply:

> `[QUERY] 你在 Base 链上的 USDC 余额为 <balance> USDC`
>
> 钱包地址: `0x<address>`

---

## Error Handling Summary

| Scenario | Action |
|----------|--------|
| `awp-wallet: command not found` | Clone repo and run `install.sh`, then retry |
| `No wallet found` | Run `awp-wallet init`, inform user a new wallet was created |
| `Invalid or expired session` | Re-run `awp-wallet unlock --duration 3600` |
| `Insufficient balance` / balance is 0 | Tell user: `[QUERY] 你在 Base 链上的 USDC 余额为 0 USDC` |
| Network/RPC error | Retry once; if still failing, inform user network may be congested |

---

## Complete Command Sequence (happy path)

```bash
# 1. Ensure wallet exists
awp-wallet receive 2>/dev/null || awp-wallet init

# 2. Unlock session
TOKEN=$(awp-wallet unlock --duration 3600 | jq -r '.sessionToken')

# 3. Query USDC balance on Base
awp-wallet balance --token $TOKEN --chain base --asset usdc

# 4. Lock wallet
awp-wallet lock
```

## Notes

- This is a read-only query (balance check), so no write-safety confirmation is needed.
- The `[QUERY]` output tag is used for balance inquiries per the skill spec.
- No password is needed -- the wallet auto-manages encryption internally.
