# BitPads

**A universal binary communication protocol — from a single heartbeat byte to a fully identified, timestamped, valued, tasked, and annotated civilisational record — in as few as one byte.**

---

## What is it?

BitPads is a binary protocol family designed from first principles at the mathematical foundation of information exchange. A single Meta byte — the first byte of every transmission — declares the frame type, content flags, and enhancement state before the receiver processes a single payload byte. The protocol scales from a one-byte heartbeat to a 44-byte fully annotated record without changing its structure.

At its core sits **BitLedger**: a binary financial and physical transmission protocol where a complete double-entry transaction encodes in **40 bits — 5 bytes**. The same wire format that carries a monetary transaction carries propellant mass, power obligations, data packets, and mission telemetry. BitPads wraps and extends that core across every transmission context, adding identity, time, task, and annotation layers that attach only when needed.

The project is implemented as a command-line tool written in hand-assembled x86-64 NASM assembly — no C runtime, no dependencies, direct kernel syscalls — producing `.bp` binary frame files that conform to the protocol specification.

---

## The Problem

Every existing protocol that handles financial or resource-flow data carries one of three fundamental burdens: **verbosity**, **schema dependency**, or **single-domain design**.

JSON and XML are off the chart — 80,000 bytes for 100 transactions where BitLedger needs 512. ISO 20022, the current banking standard, is primarily XML; its binary encodings exist but are not widely deployed and remain verbose by design. NACHA ACH files are text-based fixed-width with no per-record integrity check and no real-time capability. ISO 8583, the nearest binary peer for financial messaging, is optimised for card-network authorization flows — not for double-entry ledger semantics, not for conservation-law enforcement, and not for cross-domain use.

None of these protocols recognise the structural identity between financial accounting and physical conservation. A factory tracking material balance, a satellite constellation settling power obligations between nodes, a supply chain enforcing that every unit leaving one station arrives at another — all of these are the same algebra. Existing standards force each domain to reinvent its own wire format. BitLedger encodes the invariant directly: for every batch, the sum of all signed flows equals zero. If it does not, the protocol knows before the application does.

---

## How it Works

Every BitPads transmission begins with a **Meta byte** that declares frame mode, content flags, and whether enhancement is active. The receiver reads this byte first and knows exactly what follows — no preamble, no sync sequence, no schema lookup.

**Four frame types exist on a single spectrum:**

| Size | Frame | Contents |
|------|-------|----------|
| 1 byte | Pure Signal | Heartbeat, ACK, status flag |
| 4 bytes | Anonymous Wave | Session-context value, no identity overhead |
| 13–29 bytes | Full Record | Identity + value + optional time, task, note |
| 22–28+ bytes | BitLedger Frame | Complete double-entry record with CRC-15 integrity |

A **Layer 1** session header (8 bytes) carries sender identity, domain, permissions, and a CRC-15 computed over the full session payload. A **Layer 2** batch header (6 bytes) declares currency, scaling, and precision defaults that all records in the session inherit. **Layer 3** is the 5-byte BitLedger record itself — value, account pair, direction, and completeness flags — with conservation enforced at encoding time.

The CLI assembles frames by populating a 256-byte context block from command-line flags, then routing to the appropriate layer and component builders. Each builder writes to a stack-allocated output buffer and returns its byte count. The result is written to a file, stdout, or discarded in dry-run mode. No heap allocation. No post-processing.

---

## Current Status

BitPads is under active development. The protocol specifications are versioned and stable. The macOS CLI (`assemblycli/`) is functional and produces conformant frames for all four frame types across all protocol layers. A Linux port (`linux_assembly_cli/`) has been assembled to ELF64 but has not yet been linked or run on a Linux machine.

**What works:** frame assembly for Pure Signal, Wave, Record, and BitLedger types; Layer 1 with CRC-15 embed; Layer 2 batch headers; Meta Byte 1 and 2 construction; value encoding across four tiers; T1S and T2 time fields; task and note blocks; C0 Enhancement Grammar signal slot presence; telegraph, compact command, and context declaration Wave categories; file, stdout, and dry-run output modes.

**Known gaps:** signal slot content bytes are declared but not yet written; the 28-byte vs 22-byte footprint discrepancy in the spec needs resolution; no formal byte-level test vectors exist; no decoder — the CLI is encode-only; `--help` output is absent.

---

## The Vision

BitPads is built on one observation: **every meaningful exchange of value between entities is a conservation law**. Money conserved. Mass conserved. Energy conserved. Data conserved. The same algebraic invariant governs double-entry accounting, Kirchhoff's current law, mass balance equations, and momentum transfer. The protocol encodes that invariant at the wire level — not as a rule the application must enforce, but as a structural property of the encoding itself.

The goal is a single protocol that works without modification from a deep-space telemetry link to a high-frequency industrial controller to an IoT sensor reporting resource consumption — and that carries financial transactions on the same wire format as physical flows, because they are the same mathematics. The C0 Enhancement Grammar extends this to rich typed signalling, binary pictography using four bits per symbol through a shared codebook, and legacy-compatible telegraph emulation that a 1960s teleprinter can receive correctly.

BitPads did not set out to be universal. It became universal because the conservation invariant is universal.

---

## Industry Context

No public standard is as lean and ledger-native as BitLedger for constrained transmission environments. The closest existing protocols each cover part of the ground:

**ISO 8583** is the nearest binary peer — compact, bitmap-driven, battle-tested in card networks and ATMs. It matches BitPads for raw density on simple payment messages but lacks native double-entry account-pair semantics, compound continuation markers, session-layer CRC, and cross-domain capability.

**NACHA ACH** and **ISO 20022** are the nearest standards for batch double-entry ledger transfers. Both are text-based or XML-based by default, designed for nightly settlement volumes rather than real-time low-bandwidth links, and carry no per-record integrity mechanism equivalent to CRC-15.

**FIX/FAST** achieves high density for market data but is trading-oriented, not general-ledger. **EDI X12 / EDIFACT** handles structured commerce at scale but is text-based and verbose. Proprietary ERP formats (SAP IDoc and equivalents) are ledger-centric and binary but closed.

BitPads + BitLedger fills the gap that none of these address: an **open, binary-first, double-entry-native protocol** with session-layer CRC integrity, a universal domain extension that covers any conserved scalar, and a transmission spectrum that runs from one byte to a fully annotated record — deployable on a satellite link, an industrial bus, or a standard IP socket without format changes.

---

## What This Is

BitPads is the outermost layer of a binary protocol family built from first principles at the mathematical foundation of information exchange. A single Meta byte declares everything that follows — mode, content type, expect flags, enhancement state — before the receiver reads a single payload byte. A transmission can be a one-byte heartbeat or a fully specified 44-byte record carrying identity, time, value, task, and annotation. The protocol scales from deep space telemetry to high-frequency industrial control without changing its structure.

At its core sits **BitLedger** — a binary financial and physical transmission protocol where a complete double-entry transaction encodes in **40 bits**. The same 40-bit wire format that carries a financial transaction carries propellant mass, power obligations, data packets, and mission status. BitPads wraps and extends that core across every transmission context.

The project spans four protocol layers, each derived from the same insight: **every meaningful exchange of value between entities is a conservation law**. Money conserved. Mass conserved. Energy conserved. Data conserved. The same algebraic invariant that governs double-entry accounting governs Kirchhoff's current law, mass balance equations, and momentum transfer in mechanical systems. The protocol encodes that invariant directly at the wire level — not as a rule the application must enforce, but as a property of the encoding itself.

---

## Protocol Layers

### BitPads Protocol v2.0
*Universal 8-bit meta layer. One byte to forty-four.*

The outermost layer. A single Meta byte declares everything that follows before the receiver reads a single payload byte. The protocol scales from a one-byte heartbeat to a fully identified, timestamped, valued, tasked, and annotated record without changing its structure.

**The transmission spectrum:**

| Size      | Type                | Contents                                     |
|-----------|---------------------|----------------------------------------------|
| 1 byte    | Pure Signal         | Heartbeat, ACK request, status flag          |
| 4 bytes   | Anonymous Value     | Session-context value, no identity overhead  |
| 13 bytes  | Minimal Full Record | Identity + value, new session                |
| 29 bytes  | Full Record         | All four components: value, time, task, note |
| 22+ bytes | Full BitLedger      | Complete double-entry record in BitPads (see below) |

**Full BitLedger component breakdown:**

| Component | Description | Bytes |
|-----------|-------------|-------|
| Meta byte 1 | Universal transmission header | 1 |
| Meta byte 2 | Extended record context | 1 |
| Layer 1 | Session init — Sender ID, permissions, domain, CRC-15 | 8 |
| Layer 2 | Batch header — currency, scaling, precision, separators, rounding balance | 6 |
| Session Config Extension | Compound mode, BL block optional, opposing convention, nesting level | 1–5 |
| Layer 3 | The BitLedger record — value, flags, accounting classification | 5 |
| **Minimum total** | | **22** |

The spec's own footprint table cites **28 bytes (224 bits)** for a full BitLedger record in BitPads. The irreducible minimum of the named components is 22 bytes; the path to 28 depends on which Session Config Extension sub-fields are treated as mandatory for a given session configuration (nesting declaration, opposing convention extension, system context block). The spec notation `28+` confirms this is a floor, not a fixed size — complexity attaches on demand.

[→ Full specification: `docs/BitPads_Protocol_v2.md`]

---

### BitPads Enhancement Sub-Protocol v2.0
*Industrial-strength signalling. Binary pictography. Nested sequences.*

The C0 Enhancement Grammar reclaims the 3 upper bits of Unicode's 32 control characters — bits that have been structurally available since Baudot's 1870 telegraph codes and never used. These 3 bits become a universal flag matrix: Priority, Acknowledge Request, Continuation. Any of the 29 agreed transmission controls can carry all three flags simultaneously, in a single byte, at any declared signal slot position in a transmission.

Thirteen signal slot positions span the full transmission structure — session boundaries, batch boundaries, component boundaries, stream boundaries. Enhanced C0 bytes occupy these declared positions. Text content occupies content positions. The two never overlap. The decoder always knows which it is reading.

**The binary pictography connection:** a stream with a declared category identity allows the receiver to decode compact nibble sequences as full semantic events through a shared codebook. Four bits per symbol. Sixteen concepts per codebook. The Sumerian accounting principle — minimal mark, rich shared context — implemented in binary at sub-byte precision.

[→ Full specification: `docs/BitPads_Enhancement_Subprotocol_v2.md`]

---

### BitLedger Protocol v3.0
*Binary financial transmission. 40 bits per transaction.*

A minimal double-entry accounting record and transmission standard. Every bit position carries defined meaning. The rules of double-entry accounting are enforced at the encoding level. Three independent error detection mechanisms operate on every record without a separate checksum field.

[→ Full specification: `docs/BitLedger_Protocol_v3.md`]

---

### BitLedger Universal Domain v1.0
*Any conserved scalar. Any engineered system.*

The financial specification generalised. The same 40-bit record that carries dollars and cents carries kilograms of propellant, watt-hours of power, data packets, contractual obligations between satellites, or service-hours owed between nodes in a robot swarm. The wire format is byte-for-byte identical. The semantic interpretation changes. The conservation invariant holds in all cases.

**What changes:** the 4-bit account pair field becomes a 16-archetype relationship matrix covering every canonical flow type between any two entities in any man-made system. Source-to-Sink. Debtor-to-Creditor. Transformation. Distribution. Aggregation. The algebra is the same. The domain is unlimited.

[→ Full specification: `docs/BitLedger_Universal_Domain.md`]

---

### Compound Mode Design Note
*Linking records into logical transactions.*

Compound mode is a session-level permission that allows the `1111` account pair code to appear as a continuation marker, linking the current record to its predecessor as part of one logical multi-leg event. Documents the design tradeoff and correct implementation.

[→ Full specification: `docs/BitLedger_CompoundMode_DesignNote.md`]

---

## Document Index

| Document | Format | Description |
|----------|--------|-------------|
| `docs/BitPads_Protocol_v2.md` | Markdown | BitPads meta-layer specification |
| `docs/BitPads_Enhancement_Subprotocol_v2.md` | Markdown | C0 enhancement grammar and signal slots |
| `docs/BitLedger_Protocol_v3.md` | Markdown | Core 40-bit financial protocol |
| `docs/BitLedger_Universal_Domain.md` | Markdown | Universal domain generalisation |
| `docs/BitLedger_CompoundMode_DesignNote.md` | Markdown | Compound mode design note |

---

## Why This Matters

### Efficiency That Compounds

```
100 transactions — format comparison:

BitLedger         ~512 bytes       ████
Fixed binary      ~3,000 bytes     ████████████
CSV               ~15,000 bytes    ████████████████████████████████████████████████████████████
JSON              ~80,000 bytes    [off the chart]
```

The reduction is structural, not compressive. No decompression step. No schema lookup. No length-prefixed string parsing. The decoder reads fixed bit positions and the record is decoded. On a link measured in bits per second this matters enormously.

### Error Detection That Goes Deeper

Three independent mechanisms on every 40-bit BitLedger record:

1. **CRC-15** on the session header — polynomial `x^15 + x + 1`, covering sender identity, domain, permissions, and all session defaults
2. **Cross-layer validation** — direction and status flags mirrored in both the value block and the accounting block; a single-bit flip in either location is immediately detectable
3. **Conservation invariant** — the batch balance check catches phantom flows, missing records, and duplicated records that byte-level CRCs miss entirely

A corrupted flow record is rejected before it contaminates system state. In a spacecraft at interplanetary distance, where a single galactic cosmic ray can flip bits in a telemetry buffer, this layered detection matters at a level that raw telemetry systems cannot match.

### A Protocol That Thinks in Conservation Laws

Every system that moves resources between entities is governed by a conservation law whether it knows it or not. A factory floor that does not track material balance accumulates unexplained losses. A satellite constellation that does not track power obligations cannot settle debts between nodes. A supply chain that does not enforce the invariant that every unit leaving one station must arrive at another is operating on faith.

BitLedger enforces conservation at the wire level. For every batch: the sum of all signed flows equals zero. If it does not, the protocol knows before the application does.

---

## Design Heritage

The protocol family draws from 5,000 years of communication engineering:

**Sumerian clay tokens (c. 3000 BCE):** The first accounting systems used compact marks whose meaning expanded through shared context — the mark was minimal, the codebook was in the reader's knowledge. BitPads binary pictography is this principle implemented in nibble streams.

**Baudot telegraph code (1870):** Five bits. Thirty-two signals. The atom of binary communication. Every subsequent standard — Murray, ASCII, ISO 6429, Unicode — preserved this 5-bit space. BitPads Enhancement Grammar reclaims the 3 upper bits that 155 years of protocol evolution left structurally available.

**Double-entry accounting (c. 1494):** Luca Pacioli formalised what Venetian merchants already knew: every transaction has two sides and the sides must balance. BitLedger enforces this invariant at the encoding level, not the application level.

**Kirchhoff's current law (1845):** The sum of currents at any node equals zero. Structurally identical to double-entry balance. BitLedger's universal domain recognises this — financial accounting and physical conservation are the same algebra applied to different quantities.

---

## Architecture Decisions Worth Noting

**No floating point anywhere.** All values encode as scaled integers using the formula `N = A × 2^S + r`. Every integer from 0 to 33,554,431 is exactly reachable with no gaps. Rounding, when it occurs, is explicit — two flag bits declare direction and the encoder algorithm chooses rounding mode by account type. A monetary value is never approximated silently.

**Domain declared in the first four bits.** By the time the receiver has read the SOH marker and three domain bits it knows whether to load financial account pairs, engineering flow archetypes, or a custom semantic layer. Everything that follows is interpreted in that context. No preamble. No sync sequence. Self-framing from bit one.

**Microservices at the wire level.** Every optional capability costs zero when absent. Signal slots, System Context extensions, Setup bytes, Time blocks, Task components — none of these inflate a transmission that does not need them. The 4-byte anonymous value Wave and the 44-byte fully-specified Record use the same Meta byte architecture. Complexity attaches on demand.

**Legacy compatible by design.** A BitPads Telegraph Emulation stream transmits bytes 0–31 as genuine C0 controls. A legacy teleprinter or terminal receiving the same byte stream sees standard controls throughout — BEL rings, FS separates files, EOT closes the transmission. A BitPads receiver reads the enhancement flags in the upper 3 bits of bytes 32–255 and decodes rich typed events from the same stream.

---

## The Numbers

```
Maximum value in a single 40-bit BitLedger record:
  ~$33.5 quadrillion  (approximately 305 × global GDP)

Maximum nodes in a session (flat Sender ID):
  4,294,967,295  (4.29 billion)

Maximum nodes in a three-level identity session (8/8/16 split):
  255 networks × 255 systems × 65,535 nodes

CRC-15 burst error detection:
  100% detection of all burst errors up to 15 bits in length

Signal slot positions in a full BitPads transmission:
  13  (P1 through P13, spanning session, batch, record, stream, and Wave layers)

C0 controls in the agreed enhancement set:
  29 unconditional  +  4 conditional  =  33 total

Protocol family documents:
  5  (BitLedger v3, Universal Domain, Compound Mode Design Note,
      BitPads v2, Enhancement Sub-Protocol v2)
```

---

## The Guiding Principle

This project began with a question: what is the minimum number of bits required to unambiguously record a double-entry accounting transaction? Not to compress an existing format. Not to abbreviate a schema. To start from the mathematical definition of what a transaction is and work forward.

The answer is 40 bits. Every bit earns its position. The structure that results — a 5-byte record that enforces conservation, carries its own error detection, and decodes without a schema — turned out to be the same structure needed for engineering telemetry, IoT resource accounting, spacecraft operations, and any other domain where quantities flow between entities and the flows must be verified.

The protocol did not set out to be universal. It became universal because the conservation invariant is universal.

BitPads wraps that universal core — giving any transmission context, identity, time, intent, and meaning, at exactly the cost of what it expresses.

---

*BitPads is under active development. Specifications are versioned. All wire format changes are logged in the Protocol Change Log (Appendix D of the Enhancement Sub-Protocol).*
