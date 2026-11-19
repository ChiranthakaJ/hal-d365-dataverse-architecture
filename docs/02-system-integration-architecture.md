# System Integration Architecture

## 1. End-to-End Integration Topology

The diagram below illustrates how HAL's customer-facing channels, Azure microservices, identity providers, external lending partners, and Microsoft Dataverse communicate using a **hybrid synchronous (REST/OData) and asynchronous event-driven (Azure Service Bus)** architecture.

* **Solid arrows (`-->`)** represent **Synchronous** request/response calls.
* **Dashed arrows (`-.->`)** represent **Asynchronous / Event-Driven** flows (Pub/Sub topics, webhooks, and background workers).

```mermaid
flowchart TB
    %% External Actors
    Customer(["B2B Buyer / Seller"])
    Staff(["HAL Compliance & Ops Staff"])
    Banks(["Partner Banks / Lenders"])

    %% Identity Providers
    subgraph Identity["Identity & Access Management"]
        Auth0["Auth0<br/>(Customer IdP)"]
        Entra["Microsoft Entra ID<br/>(Staff IdP & Service Principals)"]
    end

    %% Customer-Facing Channels
    subgraph Channels["Customer-Facing Channels"]
        Magento["Magento Storefront"]
        FinPortal["Financing Portal"]
    end

    %% Azure Microservices & Messaging Layer
    subgraph Azure["Azure Cloud (AKS / Container Apps & Integration Services)"]
        APIM["Azure API Management /<br/>CRM API Facade"]
        ProfileSvc["Profile Service"]
        SupportSvc["Supporting Services<br/>(Docs, Notifications, OTP, Templates)"]
        Blob[("Azure Blob Storage<br/>(KYB & Loan Binaries)")]
        SB[["Azure Service Bus<br/>(Topics & Subscriptions)"]]
        Func["Azure Functions /<br/>Integration Workers"]
    end

    %% Core CRM Platform
    subgraph D365["Microsoft Power Platform & Dynamics 365"]
        MDA["D365 Model-Driven App<br/>(Replaces Admin Backoffice)"]
        Dataverse[("Microsoft Dataverse<br/>(System of Record)")]
    end

    %% Authentication Flows
    Customer -->|"OIDC / OAuth 2.0 (JWT)"| Auth0
    Staff -->|"OIDC / OAuth 2.0 (SSO)"| Entra
    Entra -.->|"Native RBAC & Security Roles"| MDA

    %% Synchronous Customer & Core Flows
    Customer -->|"HTTPS / GraphQL & REST"| Magento
    Customer -->|"HTTPS / REST"| FinPortal
    Magento -->|"Sync REST (JSON / Auth0 JWT)"| ProfileSvc
    ProfileSvc -->|"Sync REST (JSON)"| APIM
    FinPortal -->|"Sync REST (JSON)"| APIM
    APIM -->|"Sync OData v4 / REST<br/>(Entra S2S JWT)"| Dataverse
    MDA -->|"Sync Native Web API"| Dataverse

    %% Supporting Services Flows
    ProfileSvc -->|"Sync REST (OTP & Doc Upload)"| SupportSvc
    FinPortal -->|"Sync REST (Loan Doc Upload)"| SupportSvc
    SupportSvc -->|"HTTPS / Azure SDK"| Blob
    SupportSvc -->|"Sync REST (Doc Metadata)"| APIM

    %% Asynchronous / Event-Driven Flows
    Dataverse -.->|"Async Domain Events<br/>(Service Endpoint / Webhook)"| SB
    Magento -.->|"Async Catalog Events<br/>(AMQP / HTTPS)"| SB
    SB -.->|"Async Pub/Sub Trigger<br/>(AMQP)"| Func
    Func -->|"Batch OData v4 Upsert"| Dataverse
    Func -.->|"Async REST (Send Alerts)"| SupportSvc
    Func -.->|"Async REST / OpenAPI (mTLS)"| Banks
    Banks -.->|"Async Webhook Callback"| APIM
```

