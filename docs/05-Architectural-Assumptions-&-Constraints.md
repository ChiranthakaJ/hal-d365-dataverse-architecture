# 05 — Architectural Assumptions & Design Decisions

## 1. Foundational Data Modeling Decisions (`HALCoreDataModel`)

### 1.1 Dual-Role `Account` Architecture (Buyer & Vendor Unified Model)
- **Decision:** Both B2B Buyers and Marketplace Vendors are modeled on the standard Dataverse **`account`** entity rather than splitting into separate custom `hal_buyer` and `hal_vendor` tables.
- **Rationale:**
  - In B2B marketplaces, commercial entities frequently operate as **both** a supplier (selling raw materials/equipment) and a buyer (procuring operational supplies).
  - Unifying on `account` preserves native Dynamics 365 Sales/Customer Service capabilities (activities, timelines, hierarchies via `parentaccountid`, primary contact via `primarycontactid`) while distinguishing roles cleanly via **`hal_businesstype`**, **`hal_isapprovedbuyer`**, and **`hal_isapprovedvendor`**.
  - Lookup relationships clearly disambiguate role context at the transaction level: `hal_buyeraccount` on `hal_marketplaceorder` and `hal_marketplaceinvoice`, versus `hal_vendor` on `hal_vendorlisting`, `hal_marketplaceorderline`, and `hal_vendorpayout`.

### 1.2 Master Product vs. Vendor Listing Separation
- **Decision:** Catalog definitions are split into **`hal_marketplaceproduct`** (Organization-owned master SKU) and **`hal_vendorlisting`** (User/Team-owned vendor-specific offer).
- **Rationale:**
  - Prevents catalog duplication when multiple vendors list identical standardized industrial SKUs.
  - Allows marketplace administrators to curate canonical taxonomy (`hal_productcategory`), descriptions, and baseline MOQs (`hal_minimumorderquantitymoq`) independently of vendor-level pricing (`hal_unitprice`), stock (`hal_availablequantity`), and vendor SKUs (`hal_vendorsku`).

### 1.3 Multi-Vendor Order Line Splitting
- **Decision:** `hal_marketplaceorder` captures the buyer-facing checkout header (`hal_buyeraccount`, `hal_buyercontact`, `hal_deliveryregion`), while **`hal_marketplaceorderline`** binds each line item to a specific **`hal_vendor`** and **`hal_vendorlisting`**.
- **Rationale:**
  - A buyer can place one consolidated B2B order containing items from three different vendors.
  - Tracking `hal_fulfillmentstatus` on `hal_marketplaceorderline` enables independent vendor fulfillment workflows, partial order deliveries, and accurate vendor-level commission/payout attribution.

### 1.4 Ownership & Security Boundary Split
- **Decision:**
  - **Organization-Owned:** `hal_region`, `hal_productcategory`, `hal_marketplaceproduct`.
  - **User or Team-Owned:** `account`, `contact`, `hal_vendorlisting`, `hal_marketplaceorder`, `hal_marketplaceorderline`, `hal_marketplaceinvoice`, `hal_vendorpayout`.
- **Rationale:**
  - Reference tables and approved master products are globally readable across all business units and portals without row-level owner overhead.
  - Transactional and commercial offer tables require granular Business Unit, Team, and Power Pages / portal table-permission isolation so vendors can only view and manage their own listings, order lines, and payouts.

---

## 2. Operational & Technical Assumptions

1. **Currency & Multi-Currency Precision:**
   - The environment base currency is configured for regional B2B operations (e.g., **AED**), with Dataverse automatic `_base` currency fields (`hal_unitprice_base`, `hal_totalorderamount_base`, `hal_totalinvoiceamount_base`, `hal_netpayoutamount_base`) enabled on all monetary columns to support cross-border GCC/international transactions via `transactioncurrencyid` and `exchangerate`.
2. **Tax & VAT Calculation Responsibility:**
   - Standard VAT (e.g., 5% UAE VAT) and Free Zone exemptions (`hal_region.hal_isfreezone`) are calculated at checkout in Adobe Commerce and validated in Dataverse prior to issuing `hal_marketplaceinvoice`.
   - `hal_vendorpayout.hal_grosssalesamount` is calculated on **ex-VAT merchandise revenue** (`hal_subtotalamount` / pre-VAT line totals) unless specific merchant-of-record tax rules dictate otherwise.
3. **Invoice-to-Payout Settlement Dependency:**
   - `hal_vendorpayout` maintains lookup references to both **`hal_marketplaceorder`** and **`hal_marketplaceinvoice`**.
   - By policy, a `hal_vendorpayout` record transitions to `Approved for Payout` only after the corresponding `hal_marketplaceinvoice` reaches `hal_paymentstatus = Paid` (or an approved credit-insured settlement milestone) and the associated `hal_marketplaceorderline` items reach `Delivered`.
4. **Alternate Key Enforcement & Data Hygiene:**
   - All external correlation IDs (`hal_magentoentityid`, `hal_magentouserid`, `hal_magentocategoryid`, `hal_magentoproductid`, `hal_marketplacesku`, `hal_magentolistingid`, `hal_magentoorderid`, `hal_magentoorderitemid`, `hal_magentoinvoiceid`) are indexed via Dataverse **Alternate Keys** and must remain non-null and unique across source systems.
5. **Auditability & Compliance Retention:**
   - Dataverse table and column auditing is enabled across all 10 tables in `HALCoreDataModel` to satisfy financial audit trails for KYB status changes (`hal_kybstatus`), pricing edits, order state transitions (`hal_orderstate`), and payout disbursements (`hal_banktransferreference`).