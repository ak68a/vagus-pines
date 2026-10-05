# Certificate Format

## On-chain record

```
serial:        GC-{YEAR}-{SEQUENCE}      # e.g. GC-2026-00847
gold_mg:       number                     # gold weight in milligrams
artwork_id:    {ARTIST}-{YEAR}-{SLUG}     # e.g. RW-2026-MAMA-001
holder:        address                    # current holder
minted_at:     timestamp                  # unix timestamp
status:        active | redeemed
repro_limits:  { medium: count }          # authorized limits per medium
repro_counts:  { medium: count }          # permanent reproductions made
repro_pending: { medium: count }          # in-flight reservations
provenance:    Transfer[]                 # transfer history
```

## Serial number format

`{COOPERATIVE_PREFIX}-{YEAR}-{SEQUENCE}`

- Cooperative prefix: 2-4 character identifier, globally unique across the federation
- Year: 4-digit issuance year
- Sequence: 5-digit zero-padded, sequential within the cooperative per year

Examples:
- `DET-2026-00001` — first certificate from the Detroit cooperative in 2026
- `ATX-2027-00342` — 342nd certificate from the Austin cooperative in 2027

## Gold weight by artwork size

| Artwork Size | Area | Gold Weight |
|---|---|---|
| Small (16×20") | 320 sq in | 0.5 g |
| Medium (24×36") | 864 sq in | 1.0 g |
| Large (36×48") | 1,728 sq in | 1.5 g |
| Oversize (48×72") | 3,456 sq in | 2.0 g |

## Transaction types

```
type CertificateTx =
  | { op: "mint_batch"; serials: CertificateRecord[] }
  | { op: "assign"; serial: string; to: string }
  | { op: "transfer"; serial: string; to: string; price?: number }
  | { op: "redeem"; serial: string }
  | { op: "reserve_repro"; artwork_id: string; medium: Medium }
  | { op: "confirm_repro"; reservation_id: string }
  | { op: "release_repro"; reservation_id: string }
  | { op: "propose"; proposal: GovernanceProposal }
  | { op: "vote"; proposal_id: string; vote: boolean }
```

## Lifecycle states

1. **Manufactured** — off-chain. Serial assigned, gold weighed.
2. **Minted** — on-chain. Batch transaction, proofs confirmed.
3. **Assigned** — on-chain. Holder updated to member address.
4. **Transferred** — on-chain. Secondary sale, provenance appended.
5. **Redeemed** — terminal. Prunes from transient chain after epoch.
