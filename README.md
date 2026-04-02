# Drift Protocol — Compromised Admin Key Vault Drain ($285M) — PoC

> **Educational Purpose Only** — This PoC is created for security research and education
> purposes only. It runs on Solana localnet, not mainnet.

## Quick Start

```bash
git clone https://github.com/NomosLabs-Security/poc-drift-trade-2026
cd poc-drift-trade-2026
solana-keygen new --no-bip39-passphrase --silent --force
yarn install
anchor build
anchor keys sync
anchor build
anchor test --skip-build
```

## Details
- **Exploit Date:** 2026-04-01
- **Chain:** Solana
- **Loss:** ~$285M
- **Technique:** Compromised admin private key — attacker drained JLP Delta Neutral, SOL Super Staking, BTC Super Staking vaults
- **Attacker Wallet (Solana):** [HkGz4KmoZ7Zmk7HN6ndJ31UJ1qZ2qgwQxgVqQwovpZES](https://solscan.io/account/HkGz4KmoZ7Zmk7HN6ndJ31UJ1qZ2qgwQxgVqQwovpZES)
- **Attacker Wallet (Ethereum):** [0xFcC47866Bd2BD3066696662dbd1C89c882105643](https://etherscan.io/address/0xFcC47866Bd2BD3066696662dbd1C89c882105643)
- **Drift Program:** [dRiftyHA39MWEi3m9aunc5MzRF1JYuBsbn6VPcn33UH](https://solscan.io/account/dRiftyHA39MWEi3m9aunc5MzRF1JYuBsbn6VPcn33UH)
- **Expected Output:** Tests demonstrating how a single compromised admin key drains $285M from JLP, stablecoin, SOL, and BTC vaults, and how multisig + caps prevent the attack
- **Full Analysis:** [NomosLabs Security Intelligence Archive](https://nomoslabs.io/archive/drift-trade-2026)

## Vulnerability

Drift Protocol was a Solana-based decentralized perpetual exchange (perps DEX) with ~$309M in total value locked across multiple vaults.

The protocol's admin authority was a **single Solana private key** controlling all privileged vault operations.

The attacker wallet (`HkGz4KmoZ7Zmk7HN6ndJ31UJ1qZ2qgwQxgVqQwovpZES`) was created 8 days before the exploit, funded with 1 SOL via Near intents, and received a $2.52 test transfer from the Drift Vault.

On April 1 at ~11:06 ET, the attacker:
1. Changed admin keys (locked out Drift team)
2. Drained JLP Delta Neutral vault — 41.7M JLP tokens (~$155M)
3. Drained SOL Super Staking vault — ~980K SOL (~$50M)
4. Drained BTC Super Staking vault — ~282 BTC (~$30M)
5. Drained stablecoins (USDC, USDS, USDT) and 15+ other token types (~$74M)
6. Swapped all to USDC via Jupiter, bridged to Ethereum
7. Accumulated 38,820 ETH (~$82M) on Ethereum exit wallet

Drift vault TVL collapsed from **$309M to $24M**. PeckShield founder confirmed: "The admin keys behind Drift were definitely leaked or compromised."

## Tests

| Test | Description |
|------|-------------|
| `Setup — Initialize Drift Protocol with $309M TVL` | Protocol + 4 vaults created |
| `Step 0 — Recon` | Attacker tests with $2.52 transfer 8 days before |
| `Step 1 — Transfer admin authority` | Admin key compromised → authority transferred instantly |
| `Step 2 — Update all vault authorities` | All 4 vault admins changed to attacker |
| `Step 3 — Drain JLP Delta Neutral ($155M)` | 41.7M JLP tokens drained |
| `Step 4 — Drain remaining vaults ($130M)` | SOL, BTC, stablecoins drained — TVL $309M → $0 |
| `Access Control — Non-admin rejected` | Wrong admin cannot transfer authority |
| `Access Control — Wrong vault admin rejected` | Cannot withdraw with wrong admin |
| `FIXED — Attacker not multisig` | Single compromised key cannot propose admin change |
| `FIXED — Timelock blocks instant execution` | 48h timelock prevents immediate admin change |
| `FIXED — Guardian cancels malicious change` | Guardian cancels attacker's proposed change |
| `FIXED — Daily cap prevents $285M drain` | $15.45M/day cap — full drain would take >18 days |
| `FIXED — Guardian emergency pause` | Guardian pauses all protocol operations |

## Architecture

```
Rust/Anchor program structure:

programs/drift-exploit/src/lib.rs
├── VULNERABLE PATH
│   ├── initialize_protocol()      — Single admin key
│   ├── create_vault()             — Vault with admin authority
│   ├── transfer_admin()           — ❌ No multisig, no timelock
│   ├── update_vault_admin()       — ❌ Instant authority change
│   └── withdraw_from_vault()      — ❌ No caps, no delay
│
└── FIXED PATH
    ├── initialize_fixed_protocol() — Multisig + guardian + caps
    ├── propose_admin_change()      — ✅ Requires multisig
    ├── execute_admin_change()      — ✅ 48h timelock
    ├── cancel_admin_change()       — ✅ Guardian can cancel
    ├── withdraw_with_cap()         — ✅ 5% of TVL daily cap
    └── pause()                     — ✅ Guardian emergency pause
```

## License

MIT — For educational use only.
