# 03 — Integration Protocol Matrix

## 1. Overview
This matrix defines every interface connecting **Adobe Commerce (Magento 2)**, **Microsoft Dataverse (`HALCoreDataModel`)**, and **Dynamics 365 Finance / ERP**, specifying transport protocols, authentication standards, payload formats, throughput SLAs, and idempotency keys.

---

## 2. Interface Catalog

| Interface ID | Interface Name | Source → Target | Trigger / Cadence | Protocol & Transport | Auth Mechanism | Idempotency / Alternate Key | SLA / Latency Target |
|---|---|---|---|---|---|---|---|
| **INT-01** | B2B Company & Signatory Registration | Adobe Commerce → Dataverse | Real-time (Event) | HTTPS REST (`POST` → Service Bus → Dataverse Web API `PATCH`) | HMAC-SHA256 (Webhook) + Entra ID OAuth 2.0 (Client Credentials) | `account.hal_magentoentityid`<br>`contact.hal_magentouserid` | $< 5\text{ seconds}$ |
| **INT-02** | KYB & Account Approval Sync | Dataverse → Adobe Commerce | Real-time (Plugin / Service Endpoint) | HTTPS REST (`PUT /V1/company/:id`) via APIM | OAuth 2.0 Bearer Token (Magento Integration Token) | `account.hal_magentoentityid` | $< 10\text{ seconds}$ |
| **INT-03** | Category Hierarchy Sync | Dataverse → Adobe Commerce | On Create / Update | HTTPS REST (`POST/PUT /V1/categories`) via APIM | OAuth 2.0 Bearer Token | `hal_productcategory.hal_magentocategoryid`<br>`hal_productcategory.hal_categorycode` | $< 30\text{ seconds}$ |
| **INT-04** | Approved Master Product Publish | Dataverse → Adobe Commerce | On `hal_approvalstatus == Approved` | HTTPS REST (`POST/PUT /V1/products`) via APIM | OAuth 2.0 Bearer Token | `hal_marketplaceproduct.hal_marketplacesku`<br>`hal_marketplaceproduct.hal_magentoproductid` | $< 15\text{ seconds}$ |
| **INT-05** | Vendor Listing & Stock Ingestion | Adobe Commerce / Vendor Portal → Dataverse | Near Real-time (Event / 5-min Micro-batch) | HTTPS REST → Service Bus → Dataverse Web API `PATCH` | Entra ID OAuth 2.0 (Client Credentials) | `hal_vendorlisting.hal_magentolistingid` | $< 15\text{ seconds}$ |
| **INT-06** | Approved Vendor Listing Activation | Dataverse → Adobe Commerce | On `hal_approvalstatus == Approved` | HTTPS REST (`PUT /V1/marketplace/offers`) via APIM | OAuth 2.0 Bearer Token | `hal_vendorlisting.hal_magentolistingid` | $< 15\text{ seconds}$ |
| **INT-07** | Marketplace Order & Line Capture | Adobe Commerce → Dataverse | Real-time (Checkout Complete) | HTTPS REST → Service Bus Topic → Azure Function (`$batch` Web API `PATCH`) | HMAC-SHA256 + Entra ID OAuth 2.0 | `hal_marketplaceorder.hal_magentoorderid`<br>`hal_marketplaceorderline.hal_magentoorderitemid` | $< 5\text{ seconds}$ |
| **INT-08** | Line Fulfillment Status Update | Dataverse / Vendor Portal → Adobe Commerce | Real-time (On `hal_fulfillmentstatus` change) | HTTPS REST (`POST /V1/order/:id/ship`) via APIM | OAuth 2.0 Bearer Token | `hal_marketplaceorderline.hal_magentoorderitemid` | $< 10\text{ seconds}$ |
| **INT-09** | Tax Invoice Creation & ERP AR Posting | Dataverse → ERP Finance & Adobe Commerce | Event-driven (Order Fulfilled / Invoiced) | HTTPS REST / OData v4 (`POST` to ERP AR Ledger + M2 Invoice API) | Entra ID Managed Identity / OAuth 2.0 | `hal_marketplaceinvoice.hal_invoicenumber`<br>`hal_marketplaceinvoice.hal_magentoinvoiceid` | $< 30\text{ seconds}$ |
| **INT-10** | Invoice Payment Settlement Confirmation | ERP Finance → Dataverse | Near Real-time (Bank Reconciliation Event) | Azure Service Bus → Dataverse Web API `PATCH` | Entra ID OAuth 2.0 (Client Credentials) | `hal_marketplaceinvoice.hal_invoicenumber` | $< 60\text{ seconds}$ |
| **INT-11** | Vendor Payout Disbursement Batch | Dataverse → ERP AP / Treasury | Scheduled (Daily / Weekly Cut-off) | SFTP (ISO 20022 `pain.001`) or HTTPS REST to ERP AP | SSH Key / PGP + Entra ID OAuth 2.0 | `hal_vendorpayout.hal_payoutreference` | Batch window ($< 15\text{ mins}$) |
| **INT-12** | Payout Bank Confirmation & Trace Update | ERP AP / Bank → Dataverse | Event-driven (Settlement Clearing) | Azure Service Bus → Dataverse Web API `PATCH` | Entra ID OAuth 2.0 (Client Credentials) | `hal_vendorpayout.hal_payoutreference` | $< 2\text{ minutes}$ |

---

## 3. Transport & Security Standards

1. **TLS & Encryption:**
   - All external and internal API traffic enforces **TLS 1.2+** (TLS 1.3 preferred).
   - Sensitive KYB identifiers (`hal_tradelicensenumber`, `hal_nationalid`, `hal_banktransferreference`) are protected in transit via HTTPS and at rest via Dataverse Transparent Data Encryption (TDE) with column-level security profiles applied to financial and PII fields.
2. **API Gateway Governance (Azure APIM):**
   - Enforces JWT validation, IP allowlisting for Adobe Commerce webhooks, payload schema validation (JSON Schema Draft-07), and Dataverse API protection limit throttling (`6,000` requests per 5-minute sliding window per service principal).
3. **Dataverse Batching (`$batch`):**
   - Multi-line order ingestion (**INT-07**) wraps the `hal_marketplaceorder` header upsert and all child `hal_marketplaceorderline` upserts inside a single Dataverse **`$batch` transactional changeset** so partial orders never persist if a lookup validation fails.