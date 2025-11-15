# Governance

- Maintainer: AdMesh Specification Council
- Proposals: open a PR with `docs/` changes and schema changes in a single commit
- Reviews: 2 maintainer approvals required
- Releases: semantic versioning as defined in `VERSIONING.md`

## Extension RFCs

AIP supports a vendor-namespaced `ext` container across every payload. Operators may ship extensions immediately under their namespace, then submit a lightweight RFC when an extension should graduate into the core spec:

1. Document the extension fields under `ext.<vendor_id>` along with expected behavior.
2. Collect at least two independent adopters or data showing ecosystem value.
3. File an RFC issue or PR summarizing motivation, schema updates, and compatibility considerations.
4. The council targets a decision within one quarterly release; accepted proposals are merged and scheduled for the next minor/major version.
