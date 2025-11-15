# 10 · Versioning & Conformance

All payloads carry `spec_version`. Breaking changes increment the major component and require a new conformance suite. Minor changes must remain backward compatible and include migration notes.

Conformance is demonstrated by running the JSON fixtures in `tests/` against the schemas in `schemas/`, then submitting attestations back to the AIP governance group.

## Extension Namespace & Promotion

AIP supports innovation without fragmentation through a controlled extension namespace. Any operator can add new fields immediately inside a dedicated `ext` object, where their custom parameters live under their own vendor ID. This gives companies full flexibility to experiment, pass proprietary data, or build advanced features without ever touching or breaking the core protocol. If an extension proves useful across the ecosystem, the operator can submit a lightweight RFC, and we evaluate it for inclusion in the next version of AIP. This model keeps the standard stable, predictable, and clean, while allowing rapid evolution on the edges — the same governance pattern used by OpenRTB, OAuth, and Kubernetes.

**Promotion workflow**

1. Implement and document fields under your vendor namespace (e.g., `ext.acme_ai`).
2. Gather evidence (adoption, metrics, interoperability impact) proving ecosystem value.
3. Open an RFC (see `GOVERNANCE.md`) describing the proposal, schema diff, and migration expectations.
4. The specification council reviews within one release cycle; accepted changes ship in the next minor or major version.
