# BitLedger

**Binary Financial Transmission Protocol — Specification v3.0**

A complete double-entry accounting record and transmission standard engineered for minimum bit footprint. A full transaction — both sides of the entry, all accounting classification, direction, settlement status, value, precision metadata, and currency — encoded in **40 bits**.

---

## What It Is

BitLedger is a binary-native protocol for recording and transmitting financial transactions. It was designed from first principles at the binary representation of accounting states, not by compressing an existing format. The result is a structure where every bit position carries defined meaning, the rules of double-entry accounting are enforced at the encoding level, and the most common transaction costs exactly 5 bytes with no extensions required.

The protocol is intended for deployment on low-power handheld hardware where storage, transmission bandwidth, and energy are primary constraints — but its efficiency makes it applicable anywhere high-volume financial data moves between systems.

---

## Why It Matters

| Format | 100 Transactions | Parsing Required | Accounting Logic Embedded |
|---|---|---|---|
| **BitLedger** | **~512 bytes** | No — fixed bit positions | **Yes — at encoding level** |
| Fixed binary struct | ~2,000–5,000 bytes | No — schema needed | No |
| CSV | ~10,000–20,000 bytes | Yes | No |
| JSON | ~50,000–200,000 bytes | Yes | No |
| XML / SOAP | ~200,000+ bytes | Yes | No |

The reduction is structural, not compressive. No decompression step is required at the receiver.

---

## Protocol Architecture

The protocol operates in three layers plus an on-demand control record:

```
┌─────────────────────────────────────────────────────────┐
│  LAYER 1 — Session Initialisation    64 bits  8 bytes   │
│  Sender identity, permissions, CRC-15 checksum          │
│  Transmitted once per session                           │
├─────────────────────────────────────────────────────────┤
│  LAYER 2 — Set B Batch Header        48 bits  6 bytes   │
│  Denomination, precision, currency, separators          │
│  Transmitted once per batch                             │
├─────────────────────────────────────────────────────────┤
│  LAYER 3 — Set A + BitLedger Record  40 bits  5 bytes   │
│  Value block (bits 1–32) + accounting block (bits 33–40)│
│  Transmitted once per transaction                       │
├─────────────────────────────────────────────────────────┤
│  CONTROL RECORD                       8 bits  1 byte    │
│  Band change, ACK, batch close, parameter update        │
│  Transmitted on demand                                  │
└─────────────────────────────────────────────────────────┘
```

---

## The 40-Bit Transaction Record

```
SET A — VALUE AND FLAGS (bits 1–32)
┌──────────────────────┬──────────┬─────────────────────────┐
│  Multiplicand        │Multiplier│ Flags (bits 26–32)      │
│  bits 1–17           │bits 18–25│ R D S V P D/C Q         │
│  (price or value)    │(quantity)│ 26 27 28 29 30 31 32    │
└──────────────────────┴──────────┴─────────────────────────┘

BITLEDGER — ACCOUNTING CLASSIFICATION (bits 33–40)
┌──────────────┬───┬───┬───┬───┐
│ Account Pair │Dir│Sts│Cmp│Ext│
│ bits 33–36   │ 37│ 38│ 39│ 40│
└──────────────┴───┴───┴───┴───┘

Flag key: R=Rounding  D=RoundDir  S=SplitOrder  V=Direction
          P=Status  D/C=Debit/Credit  Q=QuantityPresent
```

### Bit 26–27 Rounding Signal

| Bits 26–27 | Meaning |
|---|---|
| `00` | Exact value — no rounding occurred |
| `10` | Rounded down — true value ≥ decoded value |
| `11` | Rounded up — true value ≤ decoded value |
| `01` | **Protocol error** — malformed record, reject |

### Account Pair Codes (bits 33–36)

| Code | Pair | Code | Pair |
|---|---|---|---|
| `0000` | Op Expense / Asset | `1000` | Asset / Liability |
| `0001` | Op Expense / Liability | `1001` | Asset / Equity |
| `0010` | Non-Op Expense / Asset | `1010` | Liability / Equity |
| `0011` | Non-Op Expense / Liability | `1011` | Asset / Asset |
| `0100` | Op Income / Asset | `1100` | Liability / Liability |
| `0101` | Op Income / Liability | `1101` | Equity / Equity |
| `0110` | Non-Op Income / Asset | `1110` | Correction / Netting |
| `0111` | Non-Op Income / Liability | `1111` | Compound Continuation |

---

## Value Encoding

The 25-bit value block encodes any integer from 0 to 33,554,431 without gaps using a split formula:

```
N  =  A × (2^S) + r

Where:
  S  =  Optimal Split from Layer 2 (default 8)
  A  =  floor(N / 2^S)    stored in upper bits
  r  =  N mod 2^S          stored in lower bits

Real Value  =  (N × Scaling Factor) / 10^DecimalPosition
```

Every integer in the range 0–33,554,431 is exactly reachable. The Scaling Factor in Layer 2 extends this to any denomination:

| Scaling Factor | Max Value at D=2 | Precision Step |
|---|---|---|
| × 1 | $335,544.31 | $0.01 |
| × 1,000 | $335,544,310.00 | $10.00 |
| × 1,000,000 | $335,544,310,000.00 | $10,000.00 |
| × 1,000,000,000 | $335,544,310,000,000.00 | $10,000,000.00 |

The absolute maximum expressible value is approximately **$33.5 quadrillion** — roughly 305 times global GDP — in a single 40-bit record.

---

## Error Detection

Three independent error detection mechanisms operate on every record without a separate checksum field:

1. **CRC-15** on Layer 1 (64-bit session header)
2. **Cross-layer flag validation**: bit 29 must equal bit 37, bit 30 must equal bit 38
3. **Invalid rounding state**: bits 26=0 and 27=1 is a detectable malformed record

---

## Compound Transactions

Multiple accounting entries forming one logical event are linked using the `1111` continuation marker:

```
Record 1:  Income / Asset  |  Completeness=1  (Partial — continuation follows)
Record 2:  1111 pair code  |  Completeness=0  (Full — compound closed)
           └── COGS entry, linked to Record 1 via Record Separator identity
```

The `1111` pair code signals the interpreter to link this record to its predecessor. All other bits in the continuation record function normally — direction, status, value, and debit/credit are all valid for the linked entry.

---

## Control Records

Control records begin with a leading `0` bit, distinguishing them from all transaction records:

```
0  [TTT]  [PPPP]
   type   payload

Type 000  Scaling Factor change  (band switch in 1 byte)
Type 001  Currency change
Type 010  Batch close with record count
Type 011  ACK / NACK
Type 100  Compound group open
Type 101  Optimal Split update
Type 110  Layer 2 short-form (all defaults, skip 48-bit header)
```

---

## Human-Readable Output

Every encoded record renders as a journal entry:

```
─────────────────────────────────────────────────────────────────
BITLEDGER JOURNAL ENTRY
Session : Acme Corporation  /  East Branch  (sub-entity 07)
Batch   : Group 01  /  Record 004  /  Currency: USD
─────────────────────────────────────────────────────────────────
DEBIT    Operational Expense          USD       1,247.50
CREDIT   Accounts Payable             USD       1,247.50
─────────────────────────────────────────────────────────────────
Description : Goods received on account. 5 units at $249.50 each.
Status      : Accrued — not yet settled
Precision   : Exact
─────────────────────────────────────────────────────────────────
Binary  : 00000000010011010 00000101 0011000 0001 0 1 0 0
Hex     : 04 D0 05 18 14
─────────────────────────────────────────────────────────────────
```

---

## Command-Line Tool

The BitLedger CLI tool is a Python implementation of the full protocol. No third-party dependencies — standard library only.

```bash
# Configure a profile
bitledger setup

# Encode a transaction interactively
bitledger encode

# Decode a received record
bitledger decode 04D00518 14

# Simulate a full sender/receiver session
bitledger simulate --profile retail --compound --enquiry
```

### Requirements

- Python 3.10 or higher
- No external dependencies

---

## Repository Contents

| File / Directory | Contents |
|---|---|
| `bitledger.py` | CLI entry point |
| `encoder.py` | Transaction encoder |
| `decoder.py` | Transaction decoder with validation |
| `control.py` | Control record handler |
| `simulator.py` | Sender/receiver session simulator |
| `formatter.py` | Journal entry and output formatter |
| `models.py` | Typed dataclass definitions |
| `setup_wizard.py` | Interactive profile configuration |
| `profiles.py` | Profile load/save/list/delete |
| `currencies.py` | Seeded 32-currency table |
| `errors.py` | Exception hierarchy |
| `rounding_report.py` | Typed − wire rounding observations and CLI aggregate text |
| `profiles/` | Saved named profile JSON files |
| `tests/` | Full test suite including round-trip and value range tests |
| `docs/BitLedger_Protocol_v3.docx` | Full protocol specification |
| `docs/BitLedger_Technical_Overview.docx` | Implementation guide |

---

## Large amounts and rounding audit (Python CLI)

Plan with **`bitledger make`** / **`suggest-sf`** (SF search defaults to **exact-first** in range **0…127**; **`--legacy-sf-search`** restores first-fit). When **`encode --amount`** would set rounding bits, pass **`--accept-rounding`**. Append **`--rounding-report`** on **`encode`** (with **`--amount`**), **`decode`** (use **`--compare-amount DECIMAL`** for \(\Delta\) totals), **`make`**, or **`check-amount`** to print **typed − wire** residuals with **SF index** and **dp**. **`make --json --rounding-report`** embeds a **`rounding_observation`** object. Details: **`cli_readme.md`**, **`project/analysis/value_encoding_scaling_factor_reference.md`**.

---

## Documentation

- **Protocol Specification v3.0** — Full bit-level specification for all three layers, control records, extension bytes, value range tables, compound transactions, CRC-15 specification, and journal entry format.

- **Technical Implementation Overview** — Python implementation guide for the CLI tool including data models, encoder/decoder algorithms, setup wizard question sequences, test strategy, and simulation design.

---

## Protocol Design Goals

- Every bit carries defined meaning
- Accounting integrity enforced at encoding level — not application level
- Default transactions cost nothing extra — the most common case uses exactly 40 bits
- 8-bit block discipline — no layer ends with unused zero-padded bits
- Self-framing session open — no preamble or sync sequence required
- Error detection layered across three independent mechanisms
- Designed to run on original low-power handheld hardware

---

## License

See `LICENSE` for terms.
