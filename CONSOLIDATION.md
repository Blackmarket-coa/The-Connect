<!-- cspell:ignore Blackstar blackmask Blackmarket Fleetbase Bitwarden Medusa blnk hawala MercurJS -->

# Consolidation Status — The-Connect

Part of the 2026-08-28 seven-repo BMC consolidation review. The canonical review — audit
verdicts, decisions, and the ordered roadmap — is `docs/REPO_CONSOLIDATION_REVIEW.md` in
`Blackmarket-coa/free-black-market`.

## This repo's verdict

- **Status: an unmodified upstream mirror of the Universal Commerce Protocol (UCP) — kept as an
  adoption reference, not developed as an internal project.** There are no Blackmarket commits
  here; treating it as in-progress work was a planning error the review corrects. Do not rebrand
  or diverge casually: upstream governs this spec (its own contributor playbook requires
  Enhancement Proposals for significant changes), so the realistic options remain
  track-as-dependency or a deliberate, full fork — the former is the decision for now.
- **"BMC Connect" today is protocol-by-extraction on the FBM side** (decision D3): the `/v1`
  marketplace layer, the `connect.js` embed, HMAC-signed `marketplace-webhooks`, and Ed25519
  `marketplace-signing`. No federation protocol gets built until a second real marketplace wants
  in; that trigger re-opens this repo's role.

## Why UCP stays interesting for that future step

| BMC surface today (FBM) | UCP concept |
| --- | --- |
| `/v1/marketplace/*` catalog + `/v1/checkout/sessions` | Catalog search/lookup + Checkout sessions |
| `connect.js` embed with publishable keys | Embedded Protocol (`ec.*` / `ep.cart.*` messages) |
| Ed25519 `marketplace-signing` envelopes | `/.well-known/ucp` profile with JWK request signing |
| Per-partner HMAC bridge credentials | UCP-Agent profile identification |
| `marketplace-webhooks` (HMAC, per-seller subscriptions) | (no server-side event spec in UCP — a gap) |

Known mismatch to carry into any adoption decision: UCP models one platform transacting with one
business — it has no marketplace-operator concept, no seller onboarding, no order listing beyond
`GET /orders/{id}`, and no payouts/commission. It is a strong federation/discovery front door and
roughly the front half of what BMC federation would need.
