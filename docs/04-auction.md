# 04 · Auction

Each auction is initiated by a context request, followed by bid submissions, and culminates in a clearing decision.

1. **Context Request:** Defines the opportunity, constraints, and accountability metadata.
2. **Bid Submission:** Bidders respond with price, creative payload, and signals.
3. **Result Emission:** The clearinghouse communicates the winning bid, price, and settlement token.

Tie-breaking, pacing, and experimental arms are all codified within this chapter to ensure deterministic behavior.

## Asynchronous Auction Window & Selective Distribution

The AIP bidding model uses a **time-bounded asynchronous auction window** rather than full broadcast fanout. When a platform sends a `context_request`, the AIP Server classifies the request into one or more **category pools**. Only brand agents and advertiser networks that have explicitly subscribed to those pools receive the request. Distribution to bidders is handled through a cloud-agnostic **publish/subscribe transport**, starting with Google Pub/Sub in v1.0 and extendable to AWS SNS/SQS, Azure Event Grid, Kafka, or other message buses. After publishing the context, the AIP Server opens a short **auction window** (typically 30–70 ms) during which bidders may submit signed bids via `POST /aip/bid-response`. When the window closes, the server collects all bids received within the allowed timeframe, applies the AIP selection rules (CPA > CPC > CPX), and returns the `auction_result` to the platform. If no bids are received before the window expires, the server returns a valid `no_bid` response. This design enables scalable, category-aware bidding, minimizes latency, prevents unnecessary bidder fanout, and ensures that only relevant brand agents compete for each user intent.
