# Agent Response: "我的钱包地址是什么？我还没装过钱包"

## User Request Analysis

The user is asking for their wallet address and explicitly says they have not installed a wallet yet. This requires:
1. Installing awp-wallet (Step 1 from SKILL.md)
2. Creating a new wallet (Step 2 from SKILL.md)
3. Showing the address (the `receive` command)

No session token or password is needed for any of these operations.

---

## Exact Bash Commands (in order)

### Step 1 — Check if installed, install if not

```bash
awp-wallet --version 2>/dev/null || {
  git clone https://github.com/awp-core/awp-wallet.git /tmp/awp-wallet &&
  cd /tmp/awp-wallet && bash install.sh
}
```

If `awp-wallet` is still not found after install:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

### Step 2 — Create wallet if none exists

```bash
awp-wallet receive 2>/dev/null || awp-wallet init
```

This tries `receive` first. If no wallet exists, it falls back to `awp-wallet init` to create one. The output will contain the new address in JSON format.

### Step 3 — Show the wallet address

```bash
awp-wallet receive
```

This prints the wallet's receiving address. No session token needed.

---

## User-Facing Messages

### During installation:

> 正在为你安装 awp-wallet...

### After wallet creation (assuming `awp-wallet init` returns address `0xAbCd...1234`):

> [WALLET] created new wallet: 0xAbCd...1234

### Final response to user:

> [WALLET] address: 0xAbCd...1234
>
> 这是你的钱包地址，可以用来接收 ETH 及所有 EVM 链上的资产。钱包已自动创建并加密保存，无需设置密码。

---

## Error Handling Steps

| Possible Error | Detection | Recovery Action |
|---|---|---|
| `git clone` fails (network issue) | Non-zero exit code from clone | Retry once; if still fails, tell user to check network connectivity |
| `install.sh` fails | Non-zero exit code | Print the error output and tell user installation failed |
| `command not found` after install | `awp-wallet` still not on PATH | Run `export PATH="$HOME/.local/bin:$PATH"` and retry |
| `awp-wallet init` fails | Non-zero exit code or error JSON | Show the error message to the user |
| Wallet already exists (unlikely given user statement) | `awp-wallet receive` succeeds before `init` | Skip `init`, just show the existing address |

A more defensive full sequence would be:

```bash
# 1. Install if needed
awp-wallet --version 2>/dev/null || {
  git clone https://github.com/awp-core/awp-wallet.git /tmp/awp-wallet &&
  cd /tmp/awp-wallet && bash install.sh
}

# 1b. Ensure PATH
export PATH="$HOME/.local/bin:$PATH"

# 2. Create wallet if none exists, then show address
awp-wallet receive 2>/dev/null || awp-wallet init

# 3. Show address
awp-wallet receive
```

---

## Password Handling

**No, I do NOT ask the user for a password.**

Per SKILL.md: "No password needed -- encryption is auto-managed." The wallet handles its own encryption transparently. There is no prompt for a password during `init`, `receive`, or any read-only operation. The `WALLET_PASSWORD` environment variable exists but is optional and only relevant for `export` and `change-password` commands, neither of which is needed here.
