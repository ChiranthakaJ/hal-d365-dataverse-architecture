```mermaid
erDiagram
    ACCOUNT ||--o{ CONTACT : "employs / authorized users (1:N)"
    ACCOUNT ||--o{ HAL_KYBAPPLICATION : "submits onboarding (1:N)"
    ACCOUNT ||--o{ HAL_FINANCINGAPPLICATION : "applies for financing (1:N)"
    ACCOUNT ||--o{ HAL_CATALOGITEM : "lists as seller (1:N)"
    
    CONTACT ||--o{ HAL_KYBAPPLICATION : "initiates / primary applicant (1:N)"
    CONTACT ||--o{ HAL_FINANCINGAPPLICATION : "submits application (1:N)"
    
    HAL_KYBAPPLICATION ||--o{ HAL_KYBDOCUMENT : "requires compliance docs (1:N)"
    HAL_KYBAPPLICATION ||--o{ HAL_KYBREVIEWLOG : "tracks audit decisions (1:N)"
    
    HAL_LENDINGPARTNER ||--o{ HAL_FINANCINGAPPLICATION : "underwrites loan / account (1:N)"
    HAL_FINANCINGAPPLICATION ||--o{ HAL_KYBDOCUMENT : "attaches supporting docs (1:N)"
    
    SYSTEMUSER ||--o{ HAL_KYBAPPLICATION : "assigned compliance officer (1:N)"
    SYSTEMUSER ||--o{ HAL_KYBREVIEWLOG : "reviewed by staff (1:N)"

    ACCOUNT {
        guid accountid PK
        string hal_tradelicensenumber UK "Alternate Key (UAE Trade License)"
        string hal_taxregistrationnumber UK "Alternate Key (UAE TRN / VAT)"
        string name "Legal Company Name"
        choice hal_companyrole "Buyer | Seller | Both"
        choice hal_kybstatus "Unverified | Pending | Approved | Rejected | Expired"
        string hal_magentocompanyid UK "Alternate Key"
        decimal hal_approvedcreditlimit "Currency (AED)"
        datetime hal_kybexpirydate
    }

    CONTACT {
        guid contactid PK
        string hal_auth0userid UK "Alternate Key (Auth0 sub)"
        guid parentcustomerid FK "Lookup -> Account"
        string fullname
        string emailaddress1 UK "Alternate Key"
        string mobilephone
        choice hal_portalrole "Admin | Buyer | Seller | Finance_Officer"
        boolean hal_isotpverified
    }

    HAL_KYBAPPLICATION {
        guid hal_kybapplicationid PK
        string hal_applicationnumber UK "Auto-number / AK (KYB-00001)"
        guid hal_accountid FK "Lookup -> Account"
        guid hal_applicantcontactid FK "Lookup -> Contact"
        guid ownerid FK "Lookup -> SystemUser / Team"
        choice hal_requestedrole "Buyer | Seller | Both"
        choice statuscode "Draft | Submitted | Under_Review | Info_Requested | Approved | Rejected"
        choice hal_riskrating "Low | Medium | High"
        string hal_rejectionreason
        datetime hal_submittedon
        datetime hal_reviewedon
    }

    HAL_KYBDOCUMENT {
        guid hal_kybdocumentid PK
        string hal_name "Document Title"
        guid hal_kybapplicationid FK "Lookup -> KYB Application (Nullable)"
        guid hal_financingapplicationid FK "Lookup -> Financing Application (Nullable)"
        choice hal_documenttype "Trade_License | Emirates_ID | Passport | Bank_Statement | VAT_Cert | MOA"
        string hal_blobstorageuri "Azure Blob Secure Pointer"
        string hal_externaldocid UK "Alternate Key (Doc Service ID)"
        choice hal_verificationstatus "Uploaded | Verified | Rejected | Expired"
        datetime hal_expirydate
    }

    HAL_KYBREVIEWLOG {
        guid hal_kybreviewlogid PK
        string hal_name "Review Checkpoint"
        guid hal_kybapplicationid FK "Lookup -> KYB Application"
        guid hal_reviewerid FK "Lookup -> SystemUser (Entra ID)"
        choice hal_action "Approved | Rejected | Escalated | Requested_More_Info"
        memo hal_comments "Compliance Officer Notes"
        datetime hal_actiontimestamp
    }

    HAL_LENDINGPARTNER {
        guid hal_lendingpartnerid PK
        string hal_partnercode UK "Alternate Key (e.g., ENBD, FAB, WIO)"
        string hal_name "Bank / Lender Name"
        boolean hal_supportsworkingcapital
        boolean hal_supportsaccountopening
        decimal hal_minfinancingamount "Currency (AED)"
        decimal hal_maxfinancingamount "Currency (AED)"
        choice statuscode "Active | Inactive"
    }

    HAL_FINANCINGAPPLICATION {
        guid hal_financingapplicationid PK
        string hal_applicationnumber UK "Auto-number / AK (FIN-00001)"
        guid hal_accountid FK "Lookup -> Account"
        guid hal_applicantcontactid FK "Lookup -> Contact"
        guid hal_lendingpartnerid FK "Lookup -> Lending Partner"
        choice hal_applicationtype "Working_Capital_Loan | Invoice_Financing | Business_Account_Opening"
        decimal hal_requestedamount "Currency (AED)"
        integer hal_tenuremonths
        string hal_partnerreferenceid UK "Alternate Key (Bank Ref #)"
        choice statuscode "Draft | Submitted_to_Bank | Underwriting | Offer_Made | Disbursed | Declined"
        decimal hal_approvedamount "Currency (AED)"
        decimal hal_interestrate
    }

    HAL_CATALOGITEM {
        guid hal_catalogitemid PK
        string hal_magentosku UK "Alternate Key"
        guid hal_selleraccountid FK "Lookup -> Account (Seller)"
        string hal_name "Product Name"
        string hal_category
        decimal hal_unitprice "Currency (AED)"
        boolean hal_isactive
        datetime hal_lastsyncedon
    }
    ```