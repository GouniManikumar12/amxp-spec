# 05 · Events & States

Every measurable action in AIP is recorded as an **Event**.
Events are how the protocol verifies that value was created — from a user seeing an ad to completing a conversion.

## Overview

Traditional ad systems rely on cookies or opaque tracking pixels.
AIP replaces those with **signed, timestamped events** that can be independently verified by all participants.

This means:
- No duplicate billing
- No fake clicks or conversions
- Transparent proof of every monetized action

## The Event Ladder

AIP defines three progressive event types — each building on the last.
Only **one unit** settles per `serve_token`.

| Event Type | Event Name | Trigger | Billing Unit | Verified By |
|-------------|------------|----------|---------------|--------------|
| **Exposure** | `cpx_exposure` | User sees an ad or recommendation | Cost per Exposure (CPX) | Platform |
| **Click** | `cpc_click` | User clicks or engages | Cost per Click (CPC) | Platform + Ad Network |
| **Conversion** | `cpa_conversion` | User completes a purchase or signup | Cost per Action (CPA) | Brand Agent + Ad Network |

Once a higher event (like CPA) is verified, lower-tier events (CPX or CPC) are not billed again.

## Event Lifecycle

```
[Auction] → [Exposure] → [Click] → [Conversion] → [Finalized]
```

Each transition is verified and timestamped.
If no further event occurs, the lifecycle ends at the last verified state.

## Event Schemas

### CPX Exposure Event

Event type: `cpx_exposure`

**Required Fields:**
- `event_type`: Always `"cpx_exposure"`
- `serve_token`: Serve token from the auction result
- `session_id`: Session identifier
- `platform_id`: Platform identifier
- `agent_id`: Winning advertiser agent identifier
- `wallet_id`: Wallet to charge for this exposure
- `pricing`: Object containing `unit` (CPX), `amount`, and `currency` (USD)
- `ts`: ISO 8601 timestamp

**Optional Fields:**
- `exposure_metadata`: Additional context (surface, position, visibility_ms)

See: `schemas/event-cpx-exposure.json`

### CPC Click Event

Event type: `cpc_click`

**Required Fields:**
- `event_type`: Always `"cpc_click"`
- `serve_token`: Serve token from the auction result
- `event_id`: Unique identifier for this click event
- `ts`: ISO 8601 timestamp

**Optional Fields:**
- `s2s`: Whether this is a server-to-server event
- `click_metadata`: Additional context (user_agent, ip_address, referrer)

See: `schemas/event-cpc-click.json`

### CPA Conversion Event

Event type: `cpa_conversion`

**Required Fields:**
- `event_type`: Always `"cpa_conversion"`
- `serve_token`: Serve token from the auction result
- `conversion_id`: Unique identifier for this conversion
- `conversion_type`: Type of conversion (signup, purchase, trial_start, demo_request, download, custom)
- `ts`: ISO 8601 timestamp

**Optional Fields:**
- `order_value_cents`: Order value in cents (for purchase conversions)
- `currency`: Currency code for order value
- `conversion_metadata`: Additional context (user_id, order_id, product_ids)

See: `schemas/event-cpa-conversion.json`

## State Machine

Ledger records transition through the following states:

| State | Description |
|-------|-------------|
| `PENDING` | Auction completed, awaiting exposure |
| `EXPOSED` | CPX exposure event received |
| `CLICKED` | CPC click event received |
| `CONVERTED` | CPA conversion event received |
| `FINALIZED` | Final billing completed |
| `REFUNDED` | Charge was refunded |

### State Transitions

```
PENDING → EXPOSED → CLICKED → CONVERTED → FINALIZED
                                         ↓
                                    REFUNDED
```

**Transition Rules:**
- `PENDING` → `EXPOSED`: When `cpx_exposure` event is received
- `EXPOSED` → `CLICKED`: When `cpc_click` event is received
- `CLICKED` → `CONVERTED`: When `cpa_conversion` event is received
- Any state → `FINALIZED`: When billing window closes and final charge is determined
- `FINALIZED` → `REFUNDED`: When a refund is issued

## Event Verification

Each event must include:
1. **Serve Token**: Links the event to a specific auction result
2. **Timestamp**: ISO 8601 format, used for deduplication and ordering
3. **Event Type**: One of `cpx_exposure`, `cpc_click`, `cpa_conversion`

### Deduplication

Events are deduplicated based on:
- `serve_token` + `event_type` combination
- For CPC events: `serve_token` + `event_id`
- For CPA events: `serve_token` + `conversion_id`

### Retry Semantics

- Events can be retried with the same identifiers
- Duplicate events (same serve_token + event_type) are ignored
- Timestamps are used to determine event order
- Out-of-order events are rejected (e.g., CPA before CPX)

## Ledger Integration

Each event updates the ledger record:

1. **CPX Exposure**: Creates or updates ledger with exposure timestamp and CPX charge
2. **CPC Click**: Updates ledger state to CLICKED, adds CPC charge
3. **CPA Conversion**: Updates ledger state to CONVERTED, adds CPA charge

The ledger maintains:
- All event timestamps
- Progressive charge updates (CPX → CPC → CPA)
- Final billing unit and amount
- Revenue share distribution

See: `schemas/ledger-record.json`

## Example Flow

1. **Auction completes**: Ledger created in `PENDING` state
2. **User sees ad**: Platform sends `cpx_exposure` event → State: `EXPOSED`
3. **User clicks**: Platform sends `cpc_click` event → State: `CLICKED`
4. **User converts**: Brand Agent sends `cpa_conversion` event → State: `CONVERTED`
5. **Billing window closes**: Ledger finalized → State: `FINALIZED`

Final charge: CPA amount (CPX and CPC charges are superseded)
