<!--
   Copyright 2026 UCP Authors

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
-->
<!-- cspell:ignore Blackmarket freeblackmarket ECaP storefronts -->

# UCP and BMC Connect

This page is maintained by the Blackmarket Coalition (BMC) fork of the UCP
repository. It maps **BMC Connect** — the commerce surface that exists in
production today on the Free Black Market (FBM) platform — onto UCP's
canonical concepts, so that a future federation or adoption decision starts
from an honest, current inventory instead of a fresh audit.

The posture (BMC consolidation decision D3) is **protocol by extraction**:
BMC Connect today is FBM's `/v1` marketplace layer, the `connect.js` embed
SDK, HMAC-signed marketplace webhooks, and Ed25519 request signing. UCP is
kept here as the **future front-door reference** — no protocol gets built
against it until a second real marketplace wants in. That trigger re-opens
this page as a work plan; until then it is a map.

## Concept mapping

Every BMC surface below is REST-only today; UCP counterparts note which
transports the spec defines for that concept (REST, MCP, A2A, Embedded).

| BMC Connect surface (FBM, today) | UCP counterpart | Notes |
| --- | --- | --- |
| *(none — partners receive docs and credentials out of band)* | Discovery: the business profile at `/.well-known/ucp` (`profile.json`) carrying the `ucp` metadata object — `ucp.version`, `ucp.capabilities[]` (reverse-domain ids such as `ucp.shopping.checkout`), `ucp.services[]`, `ucp.payment_handlers[]`, and the JWK signing keys | Adoption would start here: publishing a profile is additive and commits BMC to nothing operational. |
| `/v1/marketplace/*` catalog reads (REST) | Catalog Search + Catalog Lookup capabilities — REST `POST /catalog/search`, `POST /catalog/lookup`, `POST /catalog/product`; MCP tools `search_catalog`, `lookup_catalog`, `get_product` | Closest fit in the whole map; both sides are stateless reads over product/variant/availability shapes. |
| `/v1/checkout/sessions` (REST) | Checkout capability (`ucp.shopping.checkout`) — REST `POST /checkout-sessions`, `GET`/`PUT /checkout-sessions/{id}`, `POST …/complete`, `POST …/cancel`; MCP tools `create_checkout` … `cancel_checkout` | UCP's `status` state machine (`incomplete` → `ready_for_complete` → `completed`, with `requires_escalation`) is richer than FBM's session states; mapping states, not fields, is the real work. |
| `connect.js` embed SDK — versioned, frozen releases (`/v2.0.0/connect.js`, SRI-pinned), `window.FBM` API, declarative `data-fbm` widgets, publishable keys | Embedded Checkout Protocol (ECP) and Embedded Cart Protocol (ECaP) — `ec.*` / `ep.cart.*` messages (`transports/embedded_message.json`), configured via `embedded_config.json` | Different philosophy: `connect.js` renders storefront widgets on third-party sites; UCP's embedded protocols define platform↔business messaging inside an embedded frame. Complementary rather than competing — a UCP adoption would sit behind the same embed. |
| Ed25519 `marketplace-signing` (detached signatures over payload digests, keys published for verification) | RFC 9421 HTTP Message Signatures + RFC 9530 Content-Digest, with JWK keys (`kty: OKP`, `crv: Ed25519`, `alg: EdDSA`; `kid` = RFC 7638 thumbprint) published in the profile | Same curve, different envelope. Re-keying is unnecessary; adopting the header format and key-directory conventions is mechanical. |
| Per-partner HMAC bridge credentials + publishable embed keys (`pk_live_…`, SHA-256 at rest, per-vendor origin allow-lists) | Platform identification via the UCP profile and `UCP-Agent` header | UCP identifies *platforms* cryptographically; it has no notion of per-vendor embed credentials — those stay a BMC-side concern under any mapping. |
| `marketplace-webhooks` — HMAC-signed, per-seller subscriptions, many event types | Order Event Webhook — business→platform order lifecycle events POSTed to the platform's `webhook_url`, signed with Standard Webhooks headers (`Webhook-Id`, `Webhook-Timestamp`, `Webhook-Signature`) | UCP **does** specify a webhook surface (an earlier version of this mapping claimed it didn't) — but a narrower one: order-scoped and platform-inbound only. BMC's per-seller subscription model and non-order event types have no UCP counterpart. |

## What UCP does not model

Carry this into any adoption decision: UCP models **one platform
transacting with one business**. It has no marketplace-operator concept, no
seller onboarding, no order listing beyond `GET /orders/{id}`, and no
payouts or commission. It is a strong federation/discovery front door —
roughly the front half of what BMC federation would need — and the back
half (operator, onboarding, settlement) would remain BMC-side regardless.

## Status and pointers

* This fork is an unmodified upstream mirror plus this page and
    `CONSOLIDATION.md`; the spec itself is governed upstream (significant
    changes require an Enhancement Proposal there).
* The BMC-side implementations live in the
    `Blackmarket-coa/free-black-market` repository: the `/v1` layer under
    `backend/src/api/v1/`, signing and webhooks under
    `backend/src/modules/marketplace-{signing,webhooks}/`, and the embed SDK
    at `storefront/public/connect.js` (releases documented in
    `docs/integrations/fbm-connect.md` and its changelog).
* The consolidation review and decision D3 are recorded in
    `docs/REPO_CONSOLIDATION_REVIEW.md` in that repository.
