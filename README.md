# BitBridge - Secure Bitcoin-Stacks Bridge Protocol

A trustless bridge protocol enabling secure Bitcoin-to-Stacks transfers with robust validation mechanisms and enterprise-grade security features.

## Overview

BitBridge facilitates cross-chain transfers between Bitcoin and Stacks networks, allowing users to lock BTC and mint equivalent `Bitcoin-btc` tokens on Stacks. The protocol emphasizes security through multi-oracle validation, recipient whitelisting, and configurable economic controls.

## Key Features

### Multi-Oracle Validation System

- Transactions require verification from authorized oracle nodes
- Flexible oracle management via bridge owner
- Protection against invalid transaction submissions

### Whitelist Management

- Controlled recipient approval system
- Prevents unauthorized address interactions
- Owner-managed allowlist updates

### Automated Fee Management

- Configurable percentage-based transaction fees
- Dynamic fee calculation during minting
- Owner-controlled fee updates (0-10% range)

### Security Protocols

- Emergency pause/unpause functionality
- Transaction nonce tracking (prevents replay attacks)
- Deposit amount limits
- Principal validation checks

## Technical Specification

### Contract Architecture

#### Core Data Structures

```clarity
;; Configuration Variables
- bridge-owner: Contract admin (initialized to deployer)
- is-bridge-paused: Emergency stop state
- total-locked-bitcoin: BTC reserve tracking
- bridge-fee-percentage: Minting fee (basis points)
- max-deposit-amount: Single transaction limit

;; Security Maps
- authorized-oracles: Approved validation nodes
- processed-transactions: Prevented replay attacks
- recipient-whitelist: Approved receiving addresses

;; Token System
- Bitcoin-btc: SIP-010 compliant wrapped BTC
```

### Function Reference

#### User Operations

**Deposit Bitcoin** (`deposit-bitcoin`)

```clarity
(define-public (deposit-bitcoin
  (btc-tx-hash (string-ascii 64))
  (amount uint)
  (recipient principal)
)
```

- Initiated by authorized oracles
- Validates Bitcoin transaction hash
- Mints `Bitcoin-btc` minus fees
- Requirements:
  - Valid 64-character Bitcoin TX hash
  - Recipient in whitelist
  - Amount ≤ max deposit limit
  - Bridge not paused

#### Oracle Management

```clarity
(define-public (add-oracle (oracle principal))
(define-public (remove-oracle (oracle principal))
```

- Owner-restricted functions
- Maintains list of authorized validation nodes

#### Administrative Controls

```clarity
(define-public (pause-bridge)
(define-public (unpause-bridge)
(define-public (update-bridge-fee (new-fee uint))
(define-public (update-max-deposit (new-max uint))
```

- Owner-only access
- Real-time parameter adjustments

### Error Codes

| Code                         | Value | Description                       |
| ---------------------------- | ----- | --------------------------------- |
| ERR-NOT-AUTHORIZED           | u1    | Unauthorized access attempt       |
| ERR-INVALID-AMOUNT           | u2    | Invalid input amount              |
| ERR-INSUFFICIENT-BALANCE     | u3    | Insufficient funds                |
| ERR-BRIDGE-PAUSED            | u4    | Bridge in paused state            |
| ERR-TX-ALREADY-PROCESSED     | u5    | Duplicate transaction detected    |
| ERR-ORACLE-VALIDATION-FAILED | u6    | Invalid oracle signature          |
| ERR-INVALID-RECIPIENT        | u7    | Unapproved destination address    |
| ERR-MAX-DEPOSIT-EXCEEDED     | u8    | Transaction exceeds deposit limit |
| ERR-INVALID-TX-HASH          | u9    | Malformed transaction hash        |

## Security Model

### Validation Workflow

1. Bitcoin transaction confirmation
2. Oracle node verification
3. On-chain validation checks:
   - TX hash validity
   - Amount verification
   - Recipient whitelist status
   - Deposit limit compliance
4. Fee calculation and token minting

### Attack Mitigations

- **Double Spend Prevention:** TX hash tracking
- **Sybil Resistance:** Oracle authorization requirements
- **Economic Limits:** Configurable deposit ceilings
- **Governance Control:** Emergency pause capability
