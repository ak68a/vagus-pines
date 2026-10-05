# Reproduction Reservation Flow

## Problem

Concurrent buyers ordering limited-edition prints against a finite reproduction count. The system must approve exactly N orders when N remain, reject the rest, and handle failures gracefully.

## Three-phase flow

### 1. Reserve (atomic, on-chain)

Check `repro_counts[medium] + repro_pending[medium] < repro_limits[medium]`. If capacity exists, publish a reservation transaction that increments `repro_pending`.

Transaction ordering resolves races: two simultaneous reservations against the last available print — one succeeds, one fails.

### 2. Fulfill (off-chain)

The reservation triggers an API call to the print fulfillment partner. The order includes artwork file, print specifications, and shipping details. The printer confirms acceptance via webhook.

### 3. Confirm or Release (on-chain)

**Confirm:** Printer webhook confirms shipment. A confirmation transaction moves the reservation from `repro_pending` to `repro_counts` permanently.

**Release:** Print fails (out of stock, bad address, payment declined). A release transaction removes the pending reservation, freeing capacity.

## State at each step

```
repro_counts:   { print: 73 }
repro_pending:  { print: 2 }
repro_limit:    { print: 100 }
available:      25  (limit - counts - pending)
```

## Timeout safety net

If the cooperative's off-chain system goes down between reserve and confirm, the pending reservation sits on-chain until timeout. Configurable window (default 72 hours) auto-releases stale reservations so capacity is not permanently locked.

## Limit increases

An artist can authorize additional reproductions beyond the original limit by submitting a governance proposal. Raising the limit requires a cooperative vote. The default is the limit set at upload.
