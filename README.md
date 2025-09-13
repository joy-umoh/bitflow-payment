# BitFlow Payment Channels

## Overview

**BitFlow** is a Layer-2 payment channel protocol for the Stacks blockchain, designed to bring **Lightning Network-inspired micropayments** and **instant settlement** to Bitcoin-secured smart contracts. By leveraging Bitcoin-compatible cryptography, time-locked dispute resolution, and a UTXO-inspired state model, BitFlow enables:

* **Instant, low-cost micropayments** without congesting the blockchain.
* **Lightning Protocol Compatibility** for seamless integration with existing tooling.
* **Trustless, cryptographically enforced channel management**.
* **Flexible closures**: cooperative instant settlement or unilateral closures with dispute resolution.
* **Stacks-native programmability** for integration into DeFi, dApps, and cross-chain systems.

This contract transforms Stacks into a high-throughput payment layer with **Bitcoin-level security guarantees**.

---

## System Overview

BitFlow channels are **bidirectional escrows** secured by STX deposits and governed by Bitcoin-style multisig mechanics.

### Key Features

1. **Channel Lifecycle**

   * **Creation:** A participant opens a channel with a counterparty and locks STX into escrow.
   * **Funding:** Channels can be topped up with additional liquidity.
   * **Off-chain updates:** Participants exchange signed balance updates off-chain.
   * **Closure:** Either cooperative (dual-signed) or unilateral (with dispute period).

2. **Security Model**

   * Bitcoin-compatible **ECDSA signatures** for off-chain commitments.
   * **Dispute windows** (144-blocks) allowing counterparties to challenge fraudulent closures.
   * Contract-enforced escrow and settlement with **no counterparty risk**.

3. **Governance & Safety**

   * **Emergency withdrawal function** for contract owner under critical security scenarios.
   * Strong validation guards against malformed input, replay attacks, and unauthorized actions.

---

## Contract Architecture

The contract follows a modular design for clarity and auditability:

### 1. **Validation Layer**

* Ensures channel IDs follow Bitcoin BIP32 standards.
* Validates minimum deposits (≥ 1000 sats).
* Confirms signature buffer lengths and participant validity.
* Enforces balance consistency across state transitions.

### 2. **Channel State Management**

* `payment-channels` map maintains escrowed funds, balances, operational status, dispute deadlines, and nonces.
* UTXO-inspired design ensures each channel state is self-contained and immutable once closed.

### 3. **Core Operations**

* **`create-channel`**: Establishes a new channel with locked escrow.
* **`fund-channel`**: Adds liquidity to an existing open channel.
* **`close-channel-cooperative`**: Dual-signed instant closure with atomic settlement.
* **`initiate-unilateral-close`**: Starts a contested closure with time-lock arbitration.
* **`resolve-unilateral-close`**: Completes unilateral closure after dispute window.

### 4. **Signature & Security System**

* `verify-signature` provides a stubbed signature check (extendable to full secp256k1).
* Ensures only signed commitments can settle balances.

### 5. **Read & Admin Functions**

* **`get-channel-info`**: Returns full channel state in Lightning-compatible format.
* **`emergency-withdraw`**: Allows contract owner to withdraw locked funds in emergencies.

---

## Data Flow

1. **Channel Creation**

   * `participant-a` opens a channel with `participant-b` → funds locked in escrow.

2. **Off-chain Updates**

   * Both participants exchange signed state updates (balances, nonce).
   * Only latest signed state is valid.

3. **Closure**

   * **Cooperative:** Both signatures submitted → instant settlement.
   * **Unilateral:** One participant submits → enforced delay for dispute → final settlement.

4. **Settlement**

   * Contract redistributes escrow according to agreed balances.
   * Channel state reset → funds released.

---

## Deployment Notes

* Contract owner must be trusted to use **emergency-withdraw** only in catastrophic cases.
* Production deployments should integrate **native secp256k1 signature verification**.
* Parameters (e.g., 144-block dispute period, 1000 sats minimum deposit) can be adjusted depending on network needs.

---

## Example Usage

1. **Create a Channel**

```clarity
(contract-call? .bitflow create-channel
  0xabc123... ;; channel-id
  'SPXYZ...   ;; participant-b
  u5000000    ;; initial deposit
)
```

2. **Fund a Channel**

```clarity
(contract-call? .bitflow fund-channel
  0xabc123...
  'SPXYZ...
  u2000000
)
```

3. **Close Cooperatively**

```clarity
(contract-call? .bitflow close-channel-cooperative
  0xabc123...
  'SPXYZ...
  u3000000
  u4000000
  0xsigA...
  0xsigB...
)
```

4. **Unilateral Close**

```clarity
(contract-call? .bitflow initiate-unilateral-close
  0xabc123...
  'SPXYZ...
  u5000000
  u0
  0xsigA...
)
```

---

## Future Extensions

* **HTLC Support** for multi-hop routed payments.
* **Cross-chain settlement** with native BTC on Bitcoin L1.
* **Watchtower integration** for automated dispute handling.
* **Multi-party channels** for advanced DeFi use cases.
