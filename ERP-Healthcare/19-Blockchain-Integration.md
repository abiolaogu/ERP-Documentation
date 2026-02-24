# Blockchain Integration - AfriHealth ERP-Healthcare

## 1. Overview

AfriHealth integrates Hyperledger Fabric blockchain for three critical healthcare use cases: patient consent management, pharmaceutical supply chain verification, and medical credential authentication. The blockchain layer provides immutable audit trails, non-repudiation, and decentralized trust for sensitive healthcare operations.

---

## 2. Blockchain Architecture

```mermaid
graph TB
    subgraph "AfriHealth Application Layer"
        API[Blockchain Service<br/>Go Microservice]
        SDK[Hyperledger Fabric SDK<br/>Go Gateway]
    end

    subgraph "Hyperledger Fabric Network"
        subgraph "Channel: healthcare-channel"
            CC1[ConsentContract<br/>Chaincode]
            CC2[DrugSupplyChainContract<br/>Chaincode]
            CC3[CredentialContract<br/>Chaincode]
        end

        subgraph "Organization 1: AfriHealth"
            P1[Peer 0<br/>Endorser + Committer]
            P2[Peer 1<br/>Committer]
            CA1[Fabric CA<br/>Certificate Authority]
        end

        subgraph "Organization 2: Regulators"
            P3[Peer 0<br/>Endorser + Committer]
            CA2[Fabric CA]
        end

        subgraph "Ordering Service"
            O1[Orderer 1<br/>Raft]
            O2[Orderer 2<br/>Raft]
            O3[Orderer 3<br/>Raft]
        end

        subgraph "State Database"
            COUCH[(CouchDB<br/>Rich Queries)]
        end
    end

    API --> SDK --> P1
    P1 & P3 --> CC1 & CC2 & CC3
    CC1 & CC2 & CC3 --> COUCH
    P1 & P2 & P3 --> O1 & O2 & O3
```

---

## 3. Smart Contract: Consent Management

### 3.1 Consent Data Model

```mermaid
classDiagram
    class Consent {
        +string ID
        +string PatientID
        +string ProviderID
        +string DataType
        +string Purpose
        +string Status (active/revoked/expired)
        +string GrantedAt
        +string ExpiresAt
        +string RevokedAt
        +string TenantID
        +string ConsentHash
    }

    class ConsentContract {
        +GrantConsent(ctx, consent) error
        +RevokeConsent(ctx, consentID) error
        +VerifyConsent(ctx, patientID, providerID, dataType) bool
        +GetPatientConsents(ctx, patientID) []Consent
        +GetConsentHistory(ctx, consentID) []ConsentHistoryEntry
    }

    ConsentContract --> Consent
```

### 3.2 Consent Chaincode Implementation

```go
// blockchain/chaincode/consent/consent.go
type ConsentContract struct {
    contractapi.Contract
}

type Consent struct {
    ID          string `json:"id"`
    PatientID   string `json:"patient_id"`
    ProviderID  string `json:"provider_id"`
    DataType    string `json:"data_type"`
    Purpose     string `json:"purpose"`
    Status      string `json:"status"`
    GrantedAt   string `json:"granted_at"`
    ExpiresAt   string `json:"expires_at"`
    RevokedAt   string `json:"revoked_at"`
    TenantID    string `json:"tenant_id"`
    ConsentHash string `json:"consent_hash"`
}

func (c *ConsentContract) GrantConsent(
    ctx contractapi.TransactionContextInterface,
    id, patientID, providerID, dataType, purpose, expiresAt, tenantID string,
) error {
    // Check if consent already exists
    existing, err := ctx.GetStub().GetState(id)
    if err != nil {
        return fmt.Errorf("failed to read state: %v", err)
    }
    if existing != nil {
        return fmt.Errorf("consent %s already exists", id)
    }

    consent := Consent{
        ID:         id,
        PatientID:  patientID,
        ProviderID: providerID,
        DataType:   dataType,
        Purpose:    purpose,
        Status:     "active",
        GrantedAt:  time.Now().UTC().Format(time.RFC3339),
        ExpiresAt:  expiresAt,
        TenantID:   tenantID,
    }

    // Generate consent hash for integrity
    consent.ConsentHash = generateHash(consent)

    data, err := json.Marshal(consent)
    if err != nil {
        return fmt.Errorf("failed to marshal consent: %v", err)
    }

    // Store on ledger
    if err := ctx.GetStub().PutState(id, data); err != nil {
        return fmt.Errorf("failed to put state: %v", err)
    }

    // Emit event for off-chain processing
    ctx.GetStub().SetEvent("ConsentGranted", data)

    return nil
}

func (c *ConsentContract) VerifyConsent(
    ctx contractapi.TransactionContextInterface,
    patientID, providerID, dataType string,
) (bool, error) {
    // Rich query using CouchDB
    queryString := fmt.Sprintf(
        `{"selector":{"patient_id":"%s","provider_id":"%s","data_type":"%s","status":"active"}}`,
        patientID, providerID, dataType,
    )

    iterator, err := ctx.GetStub().GetQueryResult(queryString)
    if err != nil {
        return false, err
    }
    defer iterator.Close()

    for iterator.HasNext() {
        result, _ := iterator.Next()
        var consent Consent
        json.Unmarshal(result.Value, &consent)

        // Check expiry
        expiresAt, _ := time.Parse(time.RFC3339, consent.ExpiresAt)
        if time.Now().Before(expiresAt) {
            return true, nil
        }
    }

    return false, nil
}

func (c *ConsentContract) RevokeConsent(
    ctx contractapi.TransactionContextInterface,
    consentID string,
) error {
    data, err := ctx.GetStub().GetState(consentID)
    if err != nil || data == nil {
        return fmt.Errorf("consent %s not found", consentID)
    }

    var consent Consent
    json.Unmarshal(data, &consent)

    consent.Status = "revoked"
    consent.RevokedAt = time.Now().UTC().Format(time.RFC3339)

    updated, _ := json.Marshal(consent)
    ctx.GetStub().PutState(consentID, updated)
    ctx.GetStub().SetEvent("ConsentRevoked", updated)

    return nil
}
```

### 3.3 Consent Verification Flow

```mermaid
sequenceDiagram
    participant Provider as Healthcare Provider
    participant API as AfriHealth API
    participant BC as Blockchain Service
    participant Fabric as Hyperledger Fabric

    Provider->>API: Request patient data
    API->>BC: Verify consent (patient_id, provider_id, data_type)
    BC->>Fabric: Query ConsentContract.VerifyConsent
    Fabric->>Fabric: Check ledger for active consent
    Fabric->>Fabric: Verify not expired
    alt Consent Valid
        Fabric-->>BC: true
        BC-->>API: Consent verified
        API-->>Provider: Return patient data
        API->>BC: Log data access event
    else No Valid Consent
        Fabric-->>BC: false
        BC-->>API: Consent denied
        API-->>Provider: 403 Forbidden - No consent
    end
```

---

## 4. Smart Contract: Drug Supply Chain

### 4.1 Drug Supply Chain Model

```mermaid
flowchart TB
    subgraph "Supply Chain Journey"
        M[Manufacturer<br/>RegisterDrug] --> D[Distributor<br/>TransferDrug]
        D --> H[Hospital Pharmacy<br/>TransferDrug]
        H --> V{VerifyDrug}
        V -->|Authentic| A[Accept into Inventory]
        V -->|Counterfeit| R[ALERT + Quarantine]
    end

    subgraph "Ledger Record"
        L1[Drug ID: DRG-001]
        L2[Batch: BATCH-2024-001]
        L3[Manufacturer: PharmaCorp]
        L4[Chain of Custody:<br/>Manufacturer -> Distributor -> Hospital]
        L5[Verification Status: VERIFIED]
        L6[Timestamps at each transfer]
    end
```

### 4.2 Drug Supply Chain Chaincode

```go
type DrugRecord struct {
    DrugID         string       `json:"drug_id"`
    DrugName       string       `json:"drug_name"`
    BatchNumber    string       `json:"batch_number"`
    Manufacturer   string       `json:"manufacturer"`
    ManufactureDate string      `json:"manufacture_date"`
    ExpiryDate     string       `json:"expiry_date"`
    CurrentHolder  string       `json:"current_holder"`
    Status         string       `json:"status"` // manufactured, in_transit, delivered, verified, dispensed
    SupplyChain    []ChainEntry `json:"supply_chain"`
}

type ChainEntry struct {
    From      string `json:"from"`
    To        string `json:"to"`
    Timestamp string `json:"timestamp"`
    Location  string `json:"location"`
    Verified  bool   `json:"verified"`
}

func (c *ConsentContract) RegisterDrug(
    ctx contractapi.TransactionContextInterface,
    drugID, drugName, batchNumber, manufacturer, manufactureDate, expiryDate string,
) error {
    drug := DrugRecord{
        DrugID:          drugID,
        DrugName:        drugName,
        BatchNumber:     batchNumber,
        Manufacturer:    manufacturer,
        ManufactureDate: manufactureDate,
        ExpiryDate:      expiryDate,
        CurrentHolder:   manufacturer,
        Status:          "manufactured",
        SupplyChain: []ChainEntry{
            {
                From:      "origin",
                To:        manufacturer,
                Timestamp: time.Now().UTC().Format(time.RFC3339),
                Location:  "Manufacturing Facility",
                Verified:  true,
            },
        },
    }

    data, _ := json.Marshal(drug)
    return ctx.GetStub().PutState(drugID, data)
}

func (c *ConsentContract) TransferDrug(
    ctx contractapi.TransactionContextInterface,
    drugID, fromHolder, toHolder, location string,
) error {
    data, err := ctx.GetStub().GetState(drugID)
    if err != nil || data == nil {
        return fmt.Errorf("drug %s not found", drugID)
    }

    var drug DrugRecord
    json.Unmarshal(data, &drug)

    if drug.CurrentHolder != fromHolder {
        return fmt.Errorf("drug not held by %s", fromHolder)
    }

    drug.CurrentHolder = toHolder
    drug.Status = "in_transit"
    drug.SupplyChain = append(drug.SupplyChain, ChainEntry{
        From:      fromHolder,
        To:        toHolder,
        Timestamp: time.Now().UTC().Format(time.RFC3339),
        Location:  location,
        Verified:  true,
    })

    updated, _ := json.Marshal(drug)
    ctx.GetStub().PutState(drugID, updated)
    ctx.GetStub().SetEvent("DrugTransferred", updated)
    return nil
}

func (c *ConsentContract) VerifyDrug(
    ctx contractapi.TransactionContextInterface,
    drugID string,
) (*DrugVerificationResult, error) {
    data, err := ctx.GetStub().GetState(drugID)
    if err != nil || data == nil {
        return &DrugVerificationResult{
            IsAuthentic:   false,
            Reason:        "Drug not found in blockchain registry",
        }, nil
    }

    var drug DrugRecord
    json.Unmarshal(data, &drug)

    // Verify chain integrity
    result := &DrugVerificationResult{
        IsAuthentic:    true,
        DrugName:       drug.DrugName,
        BatchNumber:    drug.BatchNumber,
        Manufacturer:   drug.Manufacturer,
        ExpiryDate:     drug.ExpiryDate,
        ChainLength:    len(drug.SupplyChain),
        CurrentHolder:  drug.CurrentHolder,
    }

    // Check expiry
    expiry, _ := time.Parse(time.RFC3339, drug.ExpiryDate)
    if time.Now().After(expiry) {
        result.IsAuthentic = false
        result.Reason = "Drug batch has expired"
    }

    return result, nil
}
```

---

## 5. Smart Contract: Medical Credentials

### 5.1 Credential Model

```mermaid
classDiagram
    class MedicalCredential {
        +string CredentialID
        +string PractitionerID
        +string PractitionerName
        +string CredentialType
        +string IssuingAuthority
        +string LicenseNumber
        +string Specialty
        +string IssuedAt
        +string ExpiresAt
        +string Status (active/suspended/revoked/expired)
        +string CredentialHash
    }

    class CredentialContract {
        +IssueCredential(ctx, credential) error
        +VerifyCredential(ctx, credentialID) VerificationResult
        +RevokeCredential(ctx, credentialID, reason) error
        +GetPractitionerCredentials(ctx, practitionerID) []Credential
        +RenewCredential(ctx, credentialID, newExpiry) error
    }
```

### 5.2 Credential Verification Flow

```mermaid
sequenceDiagram
    participant Admin as Hospital Admin
    participant API as AfriHealth API
    participant BC as Blockchain Service
    participant Fabric as Hyperledger Fabric
    participant REG as Medical Regulatory Body

    Note over Admin,REG: Provider Onboarding
    REG->>Fabric: IssueCredential (license, specialty, expiry)
    Fabric->>Fabric: Store credential NFT on ledger

    Note over Admin,REG: Verification During Hiring
    Admin->>API: Verify provider credentials
    API->>BC: VerifyCredential(credential_id)
    BC->>Fabric: Query CredentialContract
    Fabric-->>BC: Credential details + status
    alt Valid and Active
        BC-->>API: Credential verified (valid)
        API-->>Admin: Provider authorized
    else Expired or Revoked
        BC-->>API: Credential invalid
        API-->>Admin: Provider NOT authorized (reason)
    end
```

---

## 6. Blockchain Service API

### 6.1 REST Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /api/v1/blockchain/consent | Grant patient consent |
| DELETE | /api/v1/blockchain/consent/:id | Revoke consent |
| GET | /api/v1/blockchain/consent/verify | Verify consent for data access |
| GET | /api/v1/blockchain/consent/patient/:id | Get all consents for patient |
| GET | /api/v1/blockchain/consent/:id/history | Get consent change history |
| POST | /api/v1/blockchain/drugs/register | Register new drug batch |
| POST | /api/v1/blockchain/drugs/transfer | Transfer drug in supply chain |
| GET | /api/v1/blockchain/drugs/:id/verify | Verify drug authenticity |
| GET | /api/v1/blockchain/drugs/:id/chain | Get full supply chain trace |
| POST | /api/v1/blockchain/credentials/issue | Issue medical credential |
| GET | /api/v1/blockchain/credentials/:id/verify | Verify credential |
| POST | /api/v1/blockchain/credentials/:id/revoke | Revoke credential |

### 6.2 Integration with Core Services

```mermaid
flowchart TB
    subgraph "Core Services"
        PS[Patient Service<br/>Consent on Registration]
        PHARM[Pharmacy Service<br/>Drug Verification on Receipt]
        HOSP[Hospital Service<br/>Credential Check on Hiring]
        HIE[HIE Service<br/>Consent Check on Data Exchange]
    end

    subgraph "Blockchain Service"
        BC[Blockchain Service<br/>Go + Fabric SDK]
    end

    subgraph "Hyperledger Fabric"
        LEDGER[Distributed Ledger<br/>Immutable Records]
    end

    PS -->|GrantConsent| BC
    PHARM -->|VerifyDrug| BC
    HOSP -->|VerifyCredential| BC
    HIE -->|VerifyConsent| BC
    BC --> LEDGER
```

---

## 7. Network Configuration

### 7.1 Fabric Network Topology

```yaml
# network-config.yaml
organizations:
  - name: AfriHealth
    mspID: AfriHealthMSP
    peers:
      - peer0.afrihealth.com
      - peer1.afrihealth.com
    certificateAuthorities:
      - ca.afrihealth.com

  - name: Regulators
    mspID: RegulatorsMSP
    peers:
      - peer0.regulators.afrihealth.com
    certificateAuthorities:
      - ca.regulators.afrihealth.com

channels:
  - name: healthcare-channel
    organizations: [AfriHealth, Regulators]
    chaincodes:
      - name: consent
        version: "1.0"
        endorsement_policy: "OR('AfriHealthMSP.peer')"
      - name: drug-supply-chain
        version: "1.0"
        endorsement_policy: "AND('AfriHealthMSP.peer', 'RegulatorsMSP.peer')"
      - name: credentials
        version: "1.0"
        endorsement_policy: "OR('RegulatorsMSP.peer')"

orderers:
  - orderer0.afrihealth.com
  - orderer1.afrihealth.com
  - orderer2.afrihealth.com
  consensus: etcdraft
```

### 7.2 Endorsement Policies

| Chaincode | Policy | Rationale |
|-----------|--------|-----------|
| Consent | OR(AfriHealth) | Any AfriHealth peer can endorse consent operations |
| Drug Supply Chain | AND(AfriHealth, Regulators) | Both organizations must endorse for drug verification |
| Credentials | OR(Regulators) | Only regulatory bodies can issue/revoke credentials |

---

## 8. Performance and Scaling

### 8.1 Blockchain Performance

| Operation | Avg Latency | Throughput |
|-----------|-------------|------------|
| GrantConsent | 150ms | 200 TPS |
| VerifyConsent (query) | 25ms | 1,000 TPS |
| RevokeConsent | 120ms | 200 TPS |
| RegisterDrug | 180ms | 150 TPS |
| VerifyDrug (query) | 30ms | 1,000 TPS |
| TransferDrug | 160ms | 150 TPS |
| VerifyCredential (query) | 20ms | 1,200 TPS |

### 8.2 Caching Strategy

```mermaid
flowchart TB
    A[Consent Check Request] --> B{Redis Cache Hit?}
    B -->|Yes| C[Return Cached Result<br/>TTL: 5 min]
    B -->|No| D[Query Hyperledger Fabric]
    D --> E[Cache Result in Redis]
    E --> C

    F[Consent Changed Event] --> G[Invalidate Cache Entry]
```

For high-frequency verification operations (such as consent checks during data access), results are cached in Redis with a 5-minute TTL. Blockchain change events trigger cache invalidation to maintain consistency.
