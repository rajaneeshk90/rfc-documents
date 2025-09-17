# Catalog Management

## Terminology
- **SKU (Stock Keeping Unit)**: Seller-assigned unique code that identifies a specific product variant (e.g., model + size + color). Example: `TSHIRT-ACME-BLUE-M`.
- **GS1 (global standards)**: Global standards organization that defines and issues the identifiers and data standards used across supply chains and retail so products, locations, and logistics items can be unambiguously identified and shared worldwide.
- **GTIN (Global Trade Item Number)**: Global identifier for a specific product (used by retailers, marketplaces, and supply chains to unambiguously identify an item). ISBNs map into GTIN too. GTIN = global, standardized; SKU = seller-internal.
- **UPC (Universal Product Code)**: The 12-digit barcode most commonly used in North America to identify a specific trade item.
- **GDSN (Global Data Synchronisation Network)**: Standards for synchronizing and linking product data and richer web-based product URIs.

## Example
1. Manufacturer issues **GTIN 0123456789012** for Acme H100 headphones (global product ID).
2. Retailer **BestShop** lists that product and assigns **SKU BS-H100-BLK** to track stock and orders.
3. Third-party seller **GadgetStore** sells the same GTIN on Amazon using their internal **SKU GAD-H100-BK**.
4. Marketplace groups offers by GTIN/ASIN on a single product page, while each seller keeps its own SKU.

## Product vs Services
- Products are itemized SKUs with stable physical attributes (GTIN/UPC/MPN, weight, dimensions, images).
- Services are offerings that often need provider metadata, schedule/availability, location/coverage, qualifications/licenses, variable pricing, and fulfillment constraints. Services may not have GTINs (but can be modeled by GTIN when monetized/invoiced).

### Differences that impact discovery & propagation
- Services need availability feeds (time slots) and provider profiles; products need inventory/price feeds.
- Dedup rules differ: provider identity + location (services) vs GTIN/title (products).
- Policy validation often includes license checks for services (e.g., electrician certificates).

## How sellers publish differential catalogs to many channels
1. **Canonical PIM/Service Catalog + channel “views”**: Keep a single canonical model (product or service) and produce per-channel views via a feed/adapter layer that transforms, filters, and enriches records for each outlet (Amazon, Google, Local Services, marketplaces, aggregator partners). PIMs (Akeneo, Salsify, Pimcore) + feed managers are standard.
2. **Channel adapters implement channel-specific mapping rules**: Required fields, local language, legal compliance, images, availability windows (for services), allowed SKUs/service types. For Amazon: Selling Partner API (Feeds and Listings). For Google: Merchant/Content APIs (Merchant API is the new replacement). These APIs support both bulk feeds and item-level updates; implement per-channel validation and error handling.

### Differential publishing patterns
- Full export for new SKUs / new services.
- Incremental/partial updates for volatile fields (price, inventory, availability, next-slot) via APIs/feed diffs to avoid resending full records and reduce propagation latency.
- Per-channel business filters (e.g., hide regulated services on Channel A, apply platform-specific pricing on Channel B).
- Enrichment layers (campaign tags, promotional flags, A/B title variants) applied only in the channel view.

## Service-specific feed & integration patterns
- **Availability feed / booking slots**: Services require a feed (or API/webhooks) that tells the channel when providers are bookable. Google Local Services & aggregators have explicit aggregator feed interfaces for providers and availability. See Google documentation for Local Services Ads Quickstart.
- **Provider objects**: Include qualifications, geo-service area, employee/contractor identity, rates, response time, cancellation rules, and verification artifacts. Aggregators often require provider-level onboarding and separate provider IDs.
- **Lead tracking & booking events**: Map the lead/booking life-cycle (lead → booked → completed) and reconcile back to the PIM/platform.

## Categories & example vendors

### PIM (Product Information Management) — canonical master data, localization, enrichment, versioning
- Akeneo — SaaS + open-source PIM focused on product catalogs and multichannel syndication.
- Salsify — PIM + product experience management (PXM) with strong marketplace/retailer connectors.
- Pimcore — open-source PIM + DAM + MDM, flexible for custom deployments.
- inRiver — enterprise PIM for complex assortments and omnichannel publishing.
- Plytix / Sales Layer — lightweight SaaS PIMs for SMBs.

### MDM (Master Data Management) / Enterprise catalog systems — master data at enterprise scale, complex hierarchies, data governance
- Stibo Systems, Informatica MDM, Riversand (now SyndigoRiversand) — for enterprises needing governance, EDI/GDSN, and multi-domain MDM.

### Feed/Marketplace Syndication & Channel Managers — transform canonical data into channel-specific formats and push updates
- Feedonomics — universal feed engine, mapping + realtime diffs + marketplace connectors.
- ChannelAdvisor — marketplace/channel management, inventory & order sync.
- Lengow, ChannelEngine, DataFeedWatch — feed transformation and channel publishing.

### GDSN / GS1 Data Pools & Product Content Networks — certified global data sync for retailers/suppliers
- 1WorldSync, Syndigo (product content syndication and GDSN support) — push GTIN-level data to retailers via GDSN.

### Headless commerce & storefront catalog services (also used as canonical for web storefronts)
- commercetools, Saleor, Shopify (Plus), BigCommerce — headless/catalog APIs and channel integrations.

### Service / booking catalog & provider platforms (for service offerings rather than physical SKUs)
- Mindbody, Rezdy, Checkfront, Bookeo — booking engines + provider catalogs, slot/availability sync.
- ServiceNow — internal service catalog/ITSM (not marketplace-facing).
- Various vertical marketplace platforms (e.g., Urban Company / Thumbtack) provide provider onboarding APIs.

## Quick vendor shortlist (widely used / market leaders)
- PIM / Product Experience (brands & retailers): Akeneo, Salsify, Pimcore.
- MDM (enterprise master data & governance): Informatica, Stibo Systems.
- Feed / marketplace/channel managers (syndication + diffs): Feedonomics, ChannelAdvisor.
- Headless / composable commerce platforms: commercetools, Shopify Plus, BigCommerce.
- Service / booking catalog platforms (services & slots): Mindbody (wellness), Rezdy (tours/activities).

## Who picks what (practical rule-of-thumb)
- **SMB / fast launch**: Akeneo (Community or Cloud) or Salsify (SaaS + PXM) for PIM; Shopify Plus or BigCommerce for commerce.
- **Mid-market**: Salsify or Pimcore + Feedonomics for syndication (good balance of integration & automation).
- **Enterprise / regulated / multi-domain**: Stibo or Informatica for MDM + a heavyweight PIM (Akeneo Enterprise, Pimcore or Salsify) + ChannelAdvisor/Feedonomics for scale.

## Resources on popular PIMs
- Akeneo: https://www.youtube.com/@AkeneoPIM/featured
- Salsify: https://www.youtube.com/channel/UCtjCfupODuRPrhLzczF0J4g
- Pimcore: https://www.youtube.com/@PimcoreOrg
