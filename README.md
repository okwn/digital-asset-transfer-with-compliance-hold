# Digital Asset Transfer with Compliance Hold

A DAML sample project implementing a compliance-gated asset transfer workflow where regulated assets cannot settle until a compliance officer explicitly approves the transfer request.

## Summary

In this model, an asset transfer is a multi-step process: the owner declares intent via a `TransferRequest`, the compliance officer reviews and records the outcome via a `ComplianceReview`, and only upon approval does settlement occur. The original `AssetHolding` is archived and a new one is created for the recipient. Rejection closes the request without settlement.

## Why This Matters

Regulated asset transfers—securities, tokens, financial instruments—require compliance verification before settlement. This includes:

- KYC/AML checks on both parties
- Sanctions and blacklist screening
- Transfer limit validation
- On-ledger audit trail of compliance decisions

DAML enforces this workflow at the ledger level. The compliance gate is not advisory; it is structural. A transfer cannot complete without explicit approval, and all compliance decisions are recorded immutably.

## Core Contracts

| Contract | Purpose |
|----------|---------|
| `AssetHolding` | Represents a digital asset held by an owner. Signatory: owner. |
| `TransferRequest` | Declares intent to transfer. Created by owner. Observer: recipient, compliance officer. |
| `ComplianceReview` | Records compliance officer's review checks. Signatory: compliance officer. |
| `CompletedTransfer` | Audit record of a settled transfer. Tri-party signatories. |
| `RejectedTransfer` | Audit record of a rejected transfer. |

## Roles

| Role | Responsibility |
|------|----------------|
| **Owner** | Holds the asset. Initiates transfer requests. |
| **Recipient** | Intended new holder. Observes request and review. |
| **ComplianceOfficer** | Reviews transfer requests. Approves or rejects. |

## Workflow

### Step 1: Initiate Transfer Request

Owner exercises `RequestTransfer` on their `AssetHolding`. The `TransferRequest` is created but the original holding remains active.

### Step 2: Record Compliance Review

Compliance officer exercises `StartComplianceReview` on the `TransferRequest`. This creates a `ComplianceReview` contract capturing the three checks and notes.

### Step 3: Compliance Decision

Compliance officer exercises one of:

- **`ApproveTransfer`**: Only valid when `kycConfirmed == True`, `transferLimitOk == True`, `blacklistCheckPassed == True`. Archives the original holding and request, creates a new holding for the recipient, and creates a `CompletedTransfer` record.
- **`RejectTransfer`**: Archives the transfer request and creates a `RejectedTransfer` record. Original holding remains untouched.

### Step 4: Settlement

On approval, the recipient holds a new `AssetHolding` with the same `assetId`, `quantity`, and `assetType`. The `CompletedTransfer` record serves as the immutable settlement record.

## Compliance Checks

The `ComplianceReview` tracks three boolean checks:

- **KYC Confirmed**: Know Your Customer verification completed for both parties
- **Transfer Limit OK**: Recipient's allowable transfer limit not exceeded
- **Blacklist Check Passed**: Neither party appears on sanctions or restricted lists

Approval requires all three to be `True`. If any is `False`, the officer must reject.

## Demo Scenarios

### Approval Scenario (Happy Path)

```
Issuer → AssetHolding (Owner)
Owner → RequestTransfer → TransferRequest
ComplianceOfficer → StartComplianceReview → ComplianceReview
ComplianceOfficer → ApproveTransfer → New AssetHolding (Recipient) + CompletedTransfer
```

**Outcome**: Recipient receives asset. `CompletedTransfer` created with tri-party signatures.

### Rejection Scenario

```
Issuer → AssetHolding (Owner)
Owner → RequestTransfer → TransferRequest
ComplianceOfficer → StartComplianceReview → ComplianceReview (kycConfirmed: false)
ComplianceOfficer → RejectTransfer → RejectedTransfer
```

**Outcome**: Original holding stays with owner. No settlement. `RejectedTransfer` created.

## Why DAML Fits This Use Case

DAML is well-suited for compliance-gated workflows because:

- **Explicit rights and roles**: Signatories and observers are explicit. No party can act outside their role.
- **Conditional workflows**: Choices enforce business rules. Approval requires all checks to pass.
- **Strong audit trail**: Every contract creation and exercise is recorded. `CompletedTransfer` and `RejectedTransfer` provide immutable records.
- **Controlled settlement logic**: Settlement happens atomically within `ApproveTransfer`. No intermediate states where the asset is in limbo.

## Showcase Phrases

- Compliance-gated transfer workflow
- Regulated asset movement
- Approval-before-settlement model
- Auditable transfer controls

## Run Instructions

```bash
daml build
daml script --dar .daml/dist/*.dar --script-name Demo:runDemo
```

## Project Structure

```
digital-asset-transfer-with-compliance-hold/
├── daml.yaml
├── README.md
└── daml/
    ├── Main.daml
    ├── Types.daml
    ├── TransferWorkflow.daml
    └── Demo.daml
```
