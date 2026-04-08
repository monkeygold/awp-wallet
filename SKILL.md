---
name: awp-wallet
description: >
  Self-custodial EVM blockchain wallet for executing on-chain operations.
  Use this skill whenever the user wants to: send or transfer crypto/tokens
  (ETH, USDC, USDT, etc.) to an address, check wallet balance or portfolio,
  get their wallet/receiving address, approve or revoke token spending
  allowances, sign messages or typed data, estimate gas costs, check
  transaction status, view transaction history, or do batch transfers.
  Supports 400+ EVM chains (Ethereum, Base, Arbitrum, Polygon, BSC, etc.)
  with automatic gasless fallback. This is the skill to use for ANY
  on-chain wallet operation — if the user mentions sending tokens,
  checking balances, wallet addresses, token approvals, or signing,
  use this skill even if they do not explicitly say "wallet".
metadata:
  openclaw:
    requires:
      bins:
        - node
        - git
        - openssl
      anyBins:
        - npm
    emoji: "\U0001F4B0"
    homepage: https://github.com/awp-core/awp-wallet
    install:
      - kind: node
        package: awp-wallet
        bins: [awp-wallet]
---

# AWP Wallet

EVM wallet for AI agents. All output is JSON. No passwords — wallet stores keys in plaintext with 0600 file permissions.

## Setup

One command creates the wallet (if needed) and returns a session token:

```bash
TOKEN=$(awp-wallet setup | jq -r '.sessionToken')
```

Without jq:
```bash
TOKEN=$(awp-wallet setup --raw)
```

### If awp-wallet is not installed

```bash
git clone https://github.com/awp-core/awp-wallet.git ~/awp-wallet && cd ~/awp-wallet && bash install.sh
```

If `command not found` after install: `export PATH="$HOME/.local/bin:$PATH"`

The installer creates the wallet automatically. Then run `awp-wallet setup --raw` to get a session token.

## Commands

`$T` = session token from setup.

### Balance / Portfolio
```bash
awp-wallet balance --token $T --chain ethereum
awp-wallet balance --token $T --chain base --asset usdc
awp-wallet portfolio --token $T
```

### Send
Confirm with user first:
```
[TX] about to send:
     to:      0xBob...1234
     amount:  50 USDC
     chain:   Base
     proceed? (y/n)
```
```bash
awp-wallet send --token $T --to 0xAddr --amount 0.1 --chain ethereum
awp-wallet send --token $T --to 0xAddr --amount 100 --asset usdc --chain base
```

### Receive (no token needed)
```bash
awp-wallet receive
```

### Approve / Revoke
```bash
awp-wallet approve --token $T --asset usdc --spender 0xRouter --amount 1000 --chain base
awp-wallet revoke  --token $T --asset usdc --spender 0xRouter --chain base
```

### Sign
```bash
awp-wallet sign-message --token $T --message "Hello World"
awp-wallet sign-typed-data --token $T --data '{"types":{...},...}'
```

### Gas Estimate
```bash
awp-wallet estimate --to 0xAddr --amount 0.1 --chain ethereum
```

### Transaction Status
```bash
awp-wallet tx-status --hash 0xHash --chain ethereum
```

### History
```bash
awp-wallet history --token $T --chain ethereum --limit 20
```

### Batch Send
```bash
awp-wallet batch --token $T --chain base \
  --ops '[{"to":"0xA","amount":"10","asset":"usdc"},{"to":"0xB","amount":"20","asset":"usdc"}]'
```

### Export (for wallet migration)
```bash
awp-wallet export                  # mnemonic
awp-wallet export-private-key      # private key
```

### Lock
```bash
awp-wallet lock
```

## Chains

`--chain` name or ID. Default: `ethereum`.

16 built-in: `ethereum` `base` `bsc` `arbitrum` `optimism` `polygon` `avalanche` `fantom` `zksync` `linea` `scroll` `mantle` `blast` `celo` `sepolia` `base-sepolia`

Custom: `--chain 99999 --rpc-url https://custom.rpc.com`

"on Base" → `--chain base`. "on BSC" / "BNB Chain" → `--chain bsc`. No chain mentioned → `ethereum`.

## Assets

`--asset` symbol or `0x...` address. Omit for native (ETH, BNB, etc.).

Built-in: `usdc` `usdt` `awp` `weth` `wbnb` `dai`

## Output Tags

| Tag | When |
|-----|------|
| `[QUERY]` | Balance, gas estimates |
| `[TX]` | Transactions — include explorer link |
| `[SIGN]` | Signing |
| `[WALLET]` | Wallet info |

## Error Recovery

| Error | Fix |
|-------|-----|
| `command not found` | Install (see Setup) |
| `No wallet found` | `awp-wallet setup` |
| `Invalid or expired session` | `awp-wallet unlock --raw` |
| `Insufficient balance` | Tell user; suggest `--mode gasless` |

## Advanced

```bash
awp-wallet chains                                       # list all chains
awp-wallet chain-info --chain zksync                    # chain details
awp-wallet wallets                                      # list wallet profiles
awp-wallet wallet-id                                    # current profile ID
awp-wallet status --token $T                            # session info
awp-wallet allowances --token $T --asset usdc --spender 0xRouter --chain base
awp-wallet verify-log                                   # audit log integrity
awp-wallet upgrade-7702 --token $T --chain ethereum     # EIP-7702 upgrade
awp-wallet revoke-7702 --token $T --chain ethereum      # revoke EIP-7702
awp-wallet deploy-4337 --token $T --chain ethereum      # smart account status
```

## Environment Variables (all optional)

| Variable | Purpose |
|----------|---------|
| `PIMLICO_API_KEY` | Enable gasless ERC-4337 transactions |
| `AWP_AGENT_ID` | Multi-agent wallet isolation |
| `AWP_SESSION_ID` | Per-session wallet isolation |

Gasless auto-activates when no native gas and `PIMLICO_API_KEY` is set. Force: `--mode gasless`.
