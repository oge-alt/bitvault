# BitVault Protocol - Bitcoin-Backed Stablecoin System

[![Clarity Version](https://img.shields.io/badge/Clarity-2.0-blue)](https://docs.stacks.co/docs/write-smart-contracts/clarity-language)

A decentralized financial primitive enabling Bitcoin-collateralized stablecoin issuance on Stacks L2, combining Bitcoin's security with programmable smart contract functionality.

## Key Features

### Core Architecture

- **Bitcoin-Native Collateralization**: Vaults secured by BTC with Stacks L2 settlement
- **SIP-010 Compliance**: Full compatibility with Stacks token standard
- **Dynamic Risk Parameters**:
  - Adjustable collateralization ratio (100-300%)
  - Configurable liquidation threshold (125% default)
  - Protocol-controlled debt ceiling

### Risk Management

- **Multi-Oracle System**: Decentralized price feed infrastructure
- **Collateral Health Monitoring**:
  ```clarity
  (define-read-only (calculate-collateral-ratio (vault-owner principal) (vault-id uint))
    (let ((vault (get-vault-details vault-owner vault-id)))
      (/ (* (get collateral-amount vault) (get-latest-btc-price))
         (get stablecoin-minted vault))
  )
  ```
- **Autonomous Liquidations**: Permissionless liquidation engine

### Protocol Economics

- **Mint/Redemption Fees**: Dynamic fee structure (50bps default)
- **Debt Ceiling Enforcement**: Global mint limit enforcement
- **Collateral Recycling**: Liquidated BTC redistribution mechanism

## Technical Specification

### Contract Parameters

| Parameter                 | Type | Default | Range    | Description               |
| ------------------------- | ---- | ------- | -------- | ------------------------- |
| `collateralization-ratio` | uint | 150%    | 100-300% | Minimum collateral ratio  |
| `liquidation-threshold`   | uint | 125%    | 110-200% | Liquidation trigger level |
| `mint-fee-bps`            | uint | 50      | 0-500    | Minting fee basis points  |
| `max-mint-limit`          | uint | 1M      | 0-10M    | Protocol debt ceiling     |

### Error Codes

| Code  | Constant                       | Description                         |
| ----- | ------------------------------ | ----------------------------------- |
| u1000 | `ERR-NOT-AUTHORIZED`           | Unauthorized governance action      |
| u1001 | `ERR-INSUFFICIENT-BALANCE`     | Vault debt exceeds collateral value |
| u1002 | `ERR-INVALID-COLLATERAL`       | Invalid BTC deposit amount          |
| u1003 | `ERR-UNDERCOLLATERALIZED`      | Collateral ratio below minimum      |
| u1004 | `ERR-ORACLE-PRICE-UNAVAILABLE` | Price feed outdated/invalid         |

## System Components

### 1. Oracle Infrastructure

- **Approved Providers**:
  ```clarity
  (define-map btc-price-oracles principal bool)
  ```
- **Price Validation**:
  - Timestamp freshness check
  - Price sanity bounds (0 < price ≤ 1,000,000,000,000)
  - Multi-oracle aggregation capability

### 2. Vault Management

**Data Structure**:

```clarity
(define-map vaults
  { owner: principal, id: uint }
  {
    collateral-amount: uint,
    stablecoin-minted: uint,
    created-at: uint
  }
)
```

**Lifecycle Operations**:

- `create-vault`: Initialize new collateral position
- `mint-stablecoin`: Generate new stablecoins against collateral
- `redeem-stablecoin`: Burn stablecoins to recover collateral
- `liquidate-vault`: Force closure of under-collateralized positions

### 3. Stablecoin Mechanics

**Supply Control**:

```clarity
(define-data-var total-supply uint u0)
```

**Token Characteristics**:
| Property | Value | Immutable |
|----------|-------|-----------|
| Name | `BitVault Protocol Token` | ✓ |
| Symbol | `BVP` | ✓ |
| Decimals | 8 | ✓ |

## Governance

### Controlled Parameters

```clarity
(define-public (update-collateralization-ratio (new-ratio uint))
(define-public (adjust-liquidation-threshold (new-threshold uint))
(define-public (modify-fee-structure (mint-fee uint) (redeem-fee uint))
```

### Oracle Management

```clarity
(define-public (add-btc-price-oracle (oracle principal))
(define-public (remove-oracle (oracle principal))
```

## Security Model

### Protocol Safeguards

1. **Collateral Verification**:

   - Minimum deposit amount enforcement
   - Real-time price validation
   - Time-decay resistant oracle data

2. **Systemic Protections**:
   ```clarity
   (asserts! (<= (+ (get stablecoin-minted vault) mint-amount)
               (var-get max-mint-limit))
   ```
3. **Operational Security**:
   - All governance actions require `CONTRACT-OWNER` privileges
   - Critical parameters have constrained adjustment ranges
   - Emergency circuit-breaker pattern implementable

## Usage Examples

### Creating a Vault

```bash
clarinet contract call bitvault create-vault u5000000
```

### Minting Stablecoins

```clarity
(contract-call? .bitvault mint-stablecoin 'SZ2J6ZY48GV1EZ5V2V5RB9MP66SW86PYKKQ9H6DPR u1 u1000000)
```

### Liquidating Position

```clarity
(contract-call? .bitvault liquidate-vault 'SP3X6QWWETNBWVGBYK6KM9R45D1E5N5FA2CDB2APV u15)
```
