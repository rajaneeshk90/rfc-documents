# Preliminary Research: Catalog Management

## Terminology
- **SKU (Stock Keeping Unit)**: Seller-assigned unique code that identifies a specific product variant (e.g., model + size + color). Example: `TSHIRT-ACME-BLUE-M`.
- **GS1 (global standards)**: Global standards organization that defines and issues identifiers and data standards across supply chains so products, locations, and logistics items can be unambiguously identified and shared worldwide.
- **GTIN (Global Trade Item Number)**: Global identifier for a specific product (used by retailers, marketplaces, and supply chains to unambiguously identify an item). ISBNs map into GTIN too. GTIN = global, standardized; SKU = seller-internal.
- **UPC (Universal Product Code)**: The 12-digit barcode most commonly used in North America to identify a specific trade item.
- **GDSN (Global Data Synchronisation Network)**: Standards and certified data pools for synchronizing and linking product data (often GTIN-level) between trading partners.

## Purpose & Scope
- Objective: Map the current landscape of catalog management for products and services across channels and identify gaps/opportunities for contribution.
- Scope: PIM, MDM, feed/syndication, headless commerce catalogs, service/booking catalogs; GTIN/SKU/GS1/GDSN standards; differential publishing patterns.
- Out of scope: Deep EDI/ERP internals, non-digital catalog print workflows, pricing science.

## Example (Identity & Offers)
1. Manufacturer issues **GTIN 0123456789012** for Acme H100 headphones (global product ID).
2. Retailer **BestShop** lists that product and assigns **SKU BS-H100-BLK** to track stock and orders.
3. Third-party seller **GadgetStore** sells the same GTIN on Amazon using internal **SKU GAD-H100-BK**.
4. Marketplace groups offers by GTIN/ASIN on one product page; each seller keeps its own SKU for operations.

## Products vs Services
- Products: Itemized SKUs with stable physical attributes (GTIN/UPC/MPN, weight, dimensions, images).
- Services: Offerings that require provider metadata, schedule/availability, location/coverage, qualifications/licenses, variable pricing, and fulfillment constraints. Services may not have GTINs (can be modeled with identifiers when monetized/invoiced).

### Differences impacting discovery & propagation
- Services need availability feeds (time slots) and provider profiles; products need inventory/price feeds.
- Dedup rules differ: provider identity + location (services) vs GTIN/title (products).
- Policy validation for services often includes license/qualification checks.

## How sellers publish differential catalogs to many channels
1. **Canonical PIM/Service Catalog + channel views**: Maintain a single canonical model and produce per-channel views via feed/adapter layers that transform, filter, and enrich records for each outlet (Amazon, Google, Local Services, marketplaces, aggregator partners).
2. **Channel adapters implement mapping rules**: Required fields, locales, legal compliance, images, availability windows (for services), allowed SKUs/service types. Amazon: Selling Partner API (Feeds/Listings). Google: Merchant/Content APIs (Merchant API replacement). Support bulk feeds and item-level updates with per-channel validation and error handling.

### Differential publishing patterns
- Full export for new SKUs/new services.
- Incremental/partial updates for volatile fields (price, inventory, availability, next-slot) via APIs/feed diffs to reduce latency.
- Per-channel business filters (e.g., hide regulated services on Channel A; channel-specific pricing on Channel B).
- Enrichment layers (campaign tags, promo flags, A/B titles) applied in channel views.

## Service-specific feed & integration patterns
- **Availability / booking slots**: Feeds or APIs/webhooks signaling when providers are bookable (e.g., Google Local Services aggregator interfaces).
- **Provider objects**: Qualifications, geo-service area, identity, rates, response times, cancellation rules, verification artifacts; often require provider-level onboarding and IDs.
- **Lead → booking lifecycle**: Track lead/booking states and reconcile back to the platform/PIM.

## Themes & Research Questions (initial)
- Theme A: Identity & Standards (GTIN, SKU, GS1, GDSN)
  - RQ: How do GTIN/SKU/GS1/GDSN standards intersect with marketplace identity and deduplication?
- Theme B: Canonical Data Stores (PIM vs MDM)
  - RQ: Where to draw the line between PIM and MDM responsibilities in mid-market vs enterprise stacks?
- Theme C: Channel Syndication & Diffs
  - RQ: What patterns yield low-latency, high-quality updates across Google/Amazon/marketplaces?
- Theme D: Services vs Products Catalogs
  - RQ: What models best represent availability and provider identity while minimizing duplication?

## Vendor landscape (categories & examples)
### PIM (Product Information Management)
- Akeneo, Salsify, Pimcore, inRiver, Plytix/Sales Layer.

### MDM (Master Data Management)
- Stibo Systems, Informatica MDM, Riversand (SyndigoRiversand).

### Feed/Marketplace Syndication & Channel Managers
- Feedonomics, ChannelAdvisor, Lengow, ChannelEngine, DataFeedWatch.

### GDSN / GS1 Data Pools & Product Content Networks
- 1WorldSync, Syndigo.

### Headless commerce & storefront catalog services
- commercetools, Saleor, Shopify (Plus), BigCommerce.

### Service / booking catalog & provider platforms
- Mindbody, Rezdy, Checkfront, Bookeo; ServiceNow (internal service catalog/ITSM).

## Selection heuristics (who picks what)
- SMB / fast launch: Akeneo (Community/Cloud) or Salsify (SaaS + PXM); Shopify Plus or BigCommerce for commerce.
- Mid-market: Salsify or Pimcore + Feedonomics for syndication.
- Enterprise / regulated / multi-domain: Stibo or Informatica for MDM + heavyweight PIM (Akeneo Enterprise, Pimcore, Salsify) + ChannelAdvisor/Feedonomics.

## Gaps & Opportunities
- Unified model for mixed catalogs (products + services) with first-class availability and identity semantics.
- Open, portable diff protocol for multi-channel updates (beyond vendor-specific feeds).

## Next Steps
- Gather 10–15 neutral sources per theme (standards, academic, integrator references).
- Build a comparison table: PIM vs MDM capabilities, governance, extensibility.
- Deep-dive: Google Merchant API vs Amazon SP-API (incremental updates).
- Draft refined RQs based on synthesis of Themes B–D.
- Interview practitioners on service availability patterns.

## Resources (seed)
- GS1: GTIN and GDSN documentation.
- Platform APIs: Google Merchant API (latest), Amazon Selling Partner API (Feeds/Listings).
- PIM/MDM docs: Akeneo, Salsify, Pimcore; Stibo, Informatica. 