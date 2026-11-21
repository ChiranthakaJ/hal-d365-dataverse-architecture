# HAL B2B Marketplace — Dynamics 365 & Dataverse Core Architecture

Enterprise Dataverse data model, solution artifacts, and integration architecture for the **HAL B2B Marketplace** platform (`HALCoreDataModel`).

---

## Repository Structure

```text
hal-d365-dataverse-architecture/
├── docs/
│   ├── 01-dataverse-entity-design.md
│   ├── 02-system-integration-architecture.md
│   ├── 03-integration-protocol-matrix.md
│   ├── 04-domain-event-contracts.md
│   └── 05-Architectural-Assumptions-&-Constraints.md
├── Exported ERD/
│   ├── HALCoreDataModel-erd.html
│   ├── HALCoreDataModel-erd.mmd
│   ├── HALCoreDataModel-erd.pdf
│   ├── HALCoreDataModel-erd.png
│   └── HALCoreDataModel-erd.svg
├── solutions/
│   ├── HALCoreDataModel_1_0_0_1_unmanaged.zip
│   └── HALCoreDataModel_1_0_0_2_managed.zip
└── README.md
```

---

## Architecture Deliverables

1. **[Dataverse Entity Design & ERD](./docs/01-dataverse-entity-design.md)** — Complete Entity-Relationship Diagram, Out-of-the-Box vs. custom table rationale, Alternate Keys, Business Process Flows, and security ownership models.
2. **[System Integration Architecture](./docs/02-system-integration-architecture.md)** — End-to-end integration diagram, integration topology connecting Dataverse with **Azure-hosted microservices** (via Azure API Management & Azure Service Bus), synchronous vs. event-driven communication matrix, identity federation (Auth0 & Entra ID), sample Azure Service Bus event contracts and ERP/financial settlement workflows.
3. **[Component Interaction & Protocol Matrix](./docs/03-integration-protocol-matrix.md)** — Detailed breakdown of protocols, authentication, synchronous vs. event-driven patterns, and exchanged data.
4. **[Domain Event Contracts](./docs/04-domain-event-contracts.md)** — Standardized Azure Service Bus JSON event schemas.
5. **[Assumptions & Constraints](./docs/05-Architectural-Assumptions-&-Constraints.md)** — Key architectural assumptions, governance guardrails, multi-currency/regional handling, and scalability constraints.

---

## Dataverse Solution Packages (`/solutions`)

Both packages are exported directly from the published **`HALCoreDataModel`** Dataverse solution (Publisher prefix: `hal_`):

* **Unmanaged Solution (Development & Source Control):**  
  [`solutions/HALCoreDataModel_1_0_0_1_unmanaged.zip`](./solutions/HALCoreDataModel_1_0_0_1_unmanaged.zip) (`v1.0.0.1`) — Use when importing into a development environment to inspect or extend table metadata, forms, views, alternate keys, and global choices.
* **Managed Solution (Test / UAT / Production Deployment):**  
  [`solutions/HALCoreDataModel_1_0_0_2_managed.zip`](./solutions/HALCoreDataModel_1_0_0_2_managed.zip) (`v1.0.0.2`) — Use for locked downstream environment deployments with managed layer protection.

### Core Domain Entities Included
* **Standard Tables:** `Account` (`account`), `Contact` (`contact`)
* **Master & Catalog Reference Tables (`Organization` ownership):** `Primary Region` (`hal_region`), `Product Category` (`hal_productcategory`), `Marketplace Product` (`hal_marketplaceproduct`)
* **Marketplace Operations & Settlement Tables (`User or team` ownership):** `Vendor Listing` (`hal_vendorlisting`), `Marketplace Order` (`hal_marketplaceorder`), `Marketplace Order Line` (`hal_marketplaceorderline`), `Marketplace Invoice` (`hal_marketplaceinvoice`), `Vendor Payout` (`hal_vendorpayout`)

---

## Exported Entity-Relationship Diagram (`/Exported ERD`)

Generated directly from published Dataverse metadata via **XrmToolBox — Dataverse ERD Visualizer**:

* **High-Resolution Vector Diagram:** [`Exported ERD/HALCoreDataModel-erd.svg`](./Exported%20ERD/HALCoreDataModel-erd.svg)
* **High-Resolution Raster Image:** [`Exported ERD/HALCoreDataModel-erd.png`](./Exported%20ERD/HALCoreDataModel-erd.png)
* **Printable Architecture Document:** [`Exported ERD/HALCoreDataModel-erd.pdf`](./Exported%20ERD/HALCoreDataModel-erd.pdf)
* **Interactive Data Dictionary:** [`Exported ERD/HALCoreDataModel-erd.html`](./Exported%20ERD/HALCoreDataModel-erd.html)
* **Mermaid Source Definition:** [`Exported ERD/HALCoreDataModel-erd.mmd`](./Exported%20ERD/HALCoreDataModel-erd.mmd)

![HALCoreDataModel Entity-Relationship Diagram](./Exported%20ERD/HALCoreDataModel-erd.svg)

---

## Deployment Instructions

1. Navigate to the [Power Apps Maker Portal](https://make.powerapps.com) and select your target Dataverse environment.
2. Open **Solutions** in the left navigation pane and select **Import solution**.
3. Browse to either `solutions/HALCoreDataModel_1_0_0_2_managed.zip` (for UAT/Production) or `solutions/HALCoreDataModel_1_0_0_1_unmanaged.zip` (for Development).
4. Complete the import wizard and confirm all **Alternate Keys** under each table's **Keys** tab show a status of **`Active`**.