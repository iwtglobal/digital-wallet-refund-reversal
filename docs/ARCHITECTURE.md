# Architecture — Digital Wallet Refund & Transaction Reversal

High-level reference for evaluating **digital wallet refund**, transaction reversal, and chargeback-style reverse posting. Educational only; vendor designs vary.

## Logical Components

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│ Support / API /  │────►│ Reverse request  │────►│ Eligibility     │
│ merchant portal  │     │ intake           │     │ + policy window │
└──────────────────┘     └────────┬─────────┘     └────────┬────────┘
                                  │                        │
                                  ▼                        ▼
                         ┌──────────────────┐     ┌─────────────────┐
                         │ Original tx      │────►│ Reverse ledger  │
                         │ linker + headroom│     │ posting engine  │
                         └──────────────────┘     └────────┬────────┘
                                                           │
                                                           ▼
                                                  ┌─────────────────┐
                                                  │ Audit + notify  │
                                                  │ + finance export│
                                                  └─────────────────┘
```

## Suggested Decision Metadata

| Field | Meaning |
|-------|---------|
| `reverse_request_id` | Idempotent ID for the refund/reversal attempt |
| `original_tx_id` | Linked original wallet posting |
| `reverse_type` | `refund` / `reversal` / `clawback` / `hold_release` |
| `requested_amount` | Amount to unwind |
| `remaining_before` | Reverseable headroom before this request |
| `decision` | `posted` / `rejected` / `pending_approval` |
| `reason_code` | Stable code for UX, ops, and finance |
| `approver_id` | Present when dual control is required |

## Audit Events Worth Keeping

- Eligibility outcome (window, status, amount headroom)  
- Link confirmation to original posting  
- Dual-control approvals for ops-initiated reverses  
- Ledger reverse post success with opposite-side accounts  
- Notification delivery to customer and merchant  
- Finance export of linked original/reverse pairs  

## Failure Modes to Design For

- Double reverse from client retries (require idempotent request IDs)  
- Partial refunds exceeding remaining reverseable amount  
- Reversing a transaction already fully unwound  
- Fee mismatch between original and reverse paths  
- Destination wallet closed or KYC-blocked on refund credit  

## Related Reading

Live product reading: [evdsystem.com](https://evdsystem.com/), [About MoboGage](https://evdsystem.com/about-mobogage/).

Platforms in the MoboGage / EVD System family keep reverse postings auditable so support can reconstruct refund and reversal chains without rewriting ledger history.
