# 04 — Domain Event Contracts (`HALCoreDataModel`)

## 1. Event Envelope Specification (CloudEvents v1.0)
All asynchronous domain events published to **Azure Service Bus** follow the **CNCF CloudEvents v1.0** JSON envelope specification to ensure uniform routing, distributed tracing, and idempotent consumption across Adobe Commerce, Dataverse, and ERP workers.

```json
{
  "specversion": "1.0",
  "id": "8f14e45f-ea9c-4b2a-91d4-2c7b90e11201",
  "source": "/hal/marketplace/dataverse",
  "type": "com.hal.marketplace.order.created.v1",
  "subject": "hal_marketplaceorder/ORD-2026-000418",
  "time": "2026-11-21T05:30:00Z",
  "datacontenttype": "application/json",
  "correlationid": "m2-chk-99817263",
  "data": {}
}
```

---

## 2. Core Domain Event Contracts

### 2.1 `com.hal.marketplace.account.kyb-approved.v1`
- **Producer:** Dataverse (`account` post-update on `hal_kybstatus` / `hal_isapprovedbuyer` / `hal_isapprovedvendor`)
- **Consumers:** Adobe Commerce (activates B2B company login / vendor storefront permissions), ERP Finance (creates AR/AP subledger customer/vendor master)

```json
{
  "accountId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "magentoEntityId": "M2-COMP-1049",
  "accountName": "Al Noor Industrial Supplies LLC",
  "businessType": "Dual",
  "kybStatus": "Verified",
  "isApprovedBuyer": true,
  "isApprovedVendor": true,
  "tradeLicenseNumber": "CN-2948172",
  "taxRegistrationNumber": "100492817400003",
  "defaultPaymentTerms": "Net 30",
  "primaryRegionCode": "AE-DXB",
  "primarySignatory": {
    "contactId": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
    "magentoUserId": "M2-USR-8821",
    "fullName": "Tariq Al Mansoori",
    "isPrimarySignatory": true
  }
}
```

---

### 2.2 `com.hal.marketplace.catalog.listing-approved.v1`
- **Producer:** Dataverse (`hal_vendorlisting` post-update when `hal_approvalstatus == Approved` and `hal_isactive == true`)
- **Consumers:** Adobe Commerce (publishes vendor offer onto product detail page)

```json
{
  "vendorListingId": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
  "magentoListingId": "M2-OFFER-55012",
  "listingTitle": "Industrial Centrifugal Pump 5HP - Al Noor Offer",
  "marketplaceProduct": {
    "marketplaceProductId": "1b9d6bcd-bbfd-4b2d-9b5d-ab8dfbbd4bed",
    "marketplaceSku": "HAL-PUMP-5HP-001",
    "magentoProductId": "M2-PROD-3301",
    "categoryCode": "IND-PUMPS"
  },
  "vendor": {
    "accountId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "magentoEntityId": "M2-COMP-1049"
  },
  "vendorSku": "AN-CP5HP-99",
  "unitPrice": 1450.00,
  "currencyCode": "AED",
  "availableQuantity": 42,
  "minimumOrderQuantity": 2,
  "isActive": true
}
```

---

### 2.3 `com.hal.marketplace.order.captured.v1`
- **Producer:** Adobe Commerce Checkout Webhook → APIM → Service Bus
- **Consumers:** Dataverse Ingestion Worker (`$batch` transactional upsert into `hal_marketplaceorder` and `hal_marketplaceorderline`)

```json
{
  "magentoOrderId": "100000418",
  "orderNumber": "ORD-2026-000418",
  "orderDate": "2026-11-21T05:28:14Z",
  "systemSource": "Adobe Commerce",
  "buyerAccountMagentoId": "M2-COMP-2088",
  "buyerContactMagentoId": "M2-USR-9104",
  "deliveryRegionCode": "AE-AUH",
  "paymentTerms": "Net 30",
  "paymentStatus": "Authorized",
  "currencyCode": "AED",
  "subtotalAmount": 4350.00,
  "vatAmount": 217.50,
  "totalOrderAmount": 4567.50,
  "lines": [
    {
      "magentoOrderItemId": "ITEM-418-1",
      "orderLineReference": "ORD-2026-000418-01",
      "marketplaceSku": "HAL-PUMP-5HP-001",
      "magentoListingId": "M2-OFFER-55012",
      "vendorMagentoEntityId": "M2-COMP-1049",
      "quantity": 3,
      "unitOfMeasure": "Piece",
      "unitPrice": 1450.00,
      "lineVatAmount": 217.50,
      "lineTotal": 4567.50
    }
  ]
}
```

---

### 2.4 `com.hal.marketplace.invoice.issued.v1`
- **Producer:** Dataverse (`hal_marketplaceinvoice` post-create/update when `hal_documentstatus == Issued`)
- **Consumers:** ERP Finance (posts AR receivable entry), Adobe Commerce (exposes invoice record in buyer account portal)

```json
{
  "marketplaceInvoiceId": "d290f1ee-6c54-4b01-90e6-d701748f0851",
  "invoiceNumber": "INV-2026-000912",
  "magentoInvoiceId": "M2-INV-912",
  "marketplaceOrderNumber": "ORD-2026-000418",
  "buyerAccount": {
    "accountId": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
    "magentoEntityId": "M2-COMP-2088",
    "taxRegistrationNumber": "100381920400003"
  },
  "invoiceDate": "2026-11-22T09:00:00Z",
  "dueDate": "2026-12-22T09:00:00Z",
  "currencyCode": "AED",
  "subtotalAmount": 4350.00,
  "vatAmount": 217.50,
  "totalInvoiceAmount": 4567.50,
  "documentStatus": "Issued",
  "paymentStatus": "Unpaid"
}
```

---

### 2.5 `com.hal.marketplace.payout.approved.v1`
- **Producer:** Dataverse (`hal_vendorpayout` post-update when `hal_paymentstatus == Approved for Payout`)
- **Consumers:** ERP AP / Treasury (executes bank disbursement and returns `hal_banktransferreference`)

```json
{
  "vendorPayoutId": "550e8400-e29b-41d4-a716-446655440000",
  "payoutReference": "PAY-2026-11-0074",
  "vendor": {
    "accountId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "magentoEntityId": "M2-COMP-1049",
    "taxRegistrationNumber": "100492817400003"
  },
  "marketplaceOrderNumber": "ORD-2026-000418",
  "marketplaceInvoiceNumber": "INV-2026-000912",
  "settlementPeriodStart": "2026-11-15T00:00:00Z",
  "settlementPeriodEnd": "2026-11-21T23:59:59Z",
  "currencyCode": "AED",
  "grossSalesAmount": 4350.00,
  "commissionRate": 8.50,
  "commissionAmount": 369.75,
  "netPayoutAmount": 3980.25,
  "paymentStatus": "Approved for Payout"
}
```