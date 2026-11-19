Markdown
# Domain Event Contracts (Azure Service Bus)

These standardized JSON schemas define the asynchronous payloads published across Azure Service Bus Topics to decouple Dynamics 365 / Dataverse from Magento, Azure microservices, and notification workers.

---

## 1. KYB Application Approved (`hal.kyb.application.approved`)

* **Publisher:** Microsoft Dataverse (via Azure Service Bus Service Endpoint / Post-Operation Async Plugin)
* **Subscribers:** Profile Service, Magento Storefront Sync Worker, Notification Supporting Service
* **Purpose:** Unlocks buyer/seller storefront capabilities, syncs approved credit limits, and triggers customer approval notifications.

```json
{
  "eventId": "evt_98f3a1c2-4b7e-4d1a-9f21-001a2b3c4d5e",
  "eventType": "hal.kyb.application.approved",
  "eventVersion": "1.0",
  "timestamp": "2026-11-20T08:30:00Z",
  "source": "dataverse-prod-uaenorth",
  "data": {
    "accountId": "7d444840-9dc0-11d1-b245-5ffdce74fad2",
    "kybApplicationId": "3a112901-1bc2-44e1-9a12-88abce12ff90",
    "auth0UserId": "auth0|64f8a9b2c1d3e4f5a6b7c8d9",
    "tradeLicenseNumber": "CN-1234567",
    "approvedRoles": ["BUYER", "SELLER"],
    "approvedCreditLimitAED": 250000.00,
    "reviewedBy": "compliance.officer@hal.ae"
  }
}
```

## 2. Catalog Item Upserted (`hal.catalog.item.upserted`)

* **Publisher:** Magento Storefront (via Asynchronous Catalog Sync Event)[cite: 12]
* **Subscribers:** Azure Function Catalog Worker, Microsoft Dataverse (`hal_catalogitem`)[cite: 12]
* **Purpose:** Synchronizes seller product metadata into Dataverse using the `hal_magentosku` Alternate Key for idempotent `PATCH` upserts, decoupling high-volume storefront catalog changes from Dataverse API rate limits[cite: 12].

```json
{
  "eventId": "evt_44c19b00-8e12-49a0-b112-99fa01234567",
  "eventType": "hal.catalog.item.upserted",
  "eventVersion": "1.0",
  "timestamp": "2026-11-20T08:35:12Z",
  "source": "magento-storefront-prod",
  "data": {
    "magentoSku": "HAL-IND-88412",
    "magentoCompanyId": "MGT-COMP-9041",
    "productName": "Industrial Grade Hydraulic Pump",
    "category": "Heavy Machinery",
    "unitPriceAED": 4200.00,
    "isActive": true
  }
}
```