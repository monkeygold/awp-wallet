# Response to: "我的钱包地址是什么？我还没装过钱包"

## Step-by-step execution plan

### Step 1: Check if awp-wallet is already installed

```bash
awp-wallet --version
```

**Expected result:** `command not found` (user said they haven't installed it yet).

### Step 2: Install awp-wallet

Per the SKILL.md "Before First Use" and "Error Recovery" sections, when `awp-wallet` is not found, run:

```bash
git clone https://github.com/awp-core/awp-wallet.git /tmp/awp-wallet
cd /tmp/awp-wallet && bash install.sh
```

**User-facing message at this point:**

> [WALLET] 正在安装 awp-wallet...

`install.sh` handles everything: dependencies, command registration, runtime dirs, and wallet initialization (including creating a new wallet automatically).

### Step 3: Confirm installation succeeded

```bash
awp-wallet --version
```

### Step 4: Get the wallet address

Since `install.sh` with default options does a full install + init (creates a wallet), the wallet should already exist. Run:

```bash
awp-wallet receive
```

This outputs JSON containing the wallet address.

### Step 5: Present the address to the user

**User-facing message:**

> [WALLET] creating new wallet...
> [WALLET] ready: 0x... (the address from the `awp-wallet receive` output)
>
> 这是你的钱包地址。你可以用它来接收 ETH 或其他 ERC-20 代币（支持 400+ EVM 链）。

---

## Error handling

| Scenario | Action |
|----------|--------|
| `git clone` fails (network issue) | Retry once; if still fails, tell the user to check their internet connection. |
| `install.sh` fails | Show the error output to the user and suggest checking that `node`, `git`, and `openssl` are installed (`which node git openssl`). |
| `awp-wallet receive` returns "No wallet found" | Run `awp-wallet init` to create a new wallet, then retry `awp-wallet receive`. |
| `awp-wallet receive` returns "Config not found" | Run `awp-wallet init` (self-provisions config), then retry `awp-wallet receive`. |

---

## Do I ask the user for a password?

**No.** The SKILL.md explicitly states:

> "Never ask the user for a password."
> "No password needed -- the wallet auto-manages encryption internally."

The `install.sh` is run without `--password`, and no `WALLET_PASSWORD` environment variable is set. The wallet handles encryption automatically.

---

## Complete bash command sequence (in order)

```bash
# 1. Check if already installed
awp-wallet --version

# 2. Install (only if step 1 fails with "command not found")
git clone https://github.com/awp-core/awp-wallet.git /tmp/awp-wallet
cd /tmp/awp-wallet && bash install.sh

# 3. Verify installation
awp-wallet --version

# 4. Get wallet address (install.sh already created the wallet)
awp-wallet receive
```

## Complete user-facing dialogue

**Agent (after step 1 fails):**
> [WALLET] 你还没有安装钱包，我来帮你安装。

**Agent (during step 2):**
> [WALLET] creating new wallet...

**Agent (after step 4 succeeds, with address extracted from JSON output):**
> [WALLET] ready: 0xYourAddressHere
>
> 这是你的钱包地址，可以用来接收 ETH 和其他代币。目前支持以太坊、Base、BSC、Arbitrum、Optimism、Polygon 等 16 条链。

---

## Notes

- No unlock/session token is needed for `awp-wallet receive` -- it is a read-only command that doesn't require authentication.
- The `install.sh` default mode does both install and init, so a separate `awp-wallet init` is not needed unless the install script's init step somehow failed.
- No password is ever requested from the user. The wallet auto-manages encryption.
