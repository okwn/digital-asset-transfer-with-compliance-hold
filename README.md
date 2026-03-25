# Digital Asset Transfer with Compliance Hold

A DAML sample project implementing a compliance-gated asset transfer workflow where regulated assets cannot settle until a compliance officer explicitly approves the transfer request.

## Summary

In this model, an asset transfer is a multi-step process: the owner declares intent via a `TransferRequest`, the compliance officer reviews and records the outcome via a `ComplianceReview`, and only upon approval does settlement occur. The original `AssetHolding` is archived and a new one is created for the recipient. Rejection or cancellation closes the request without settlement.

## Why This Matters

Regulated asset transfers—securities, tokens, financial instruments—require compliance verification before settlement. This includes:

- KYC/AML checks on both parties
- Sanctions and blacklist screening
- Transfer limit validation
- On-ledger audit trail of compliance decisions

DAML enforces this workflow at the ledger level. The compliance gate is not advisory; it is structural. A transfer cannot complete without explicit approval, and all compliance decisions are recorded immutably.

## Core Contracts

| Contract | Module | Purpose |
|----------|--------|---------|
| `AssetHolding` | Asset | Represents a digital asset held by an owner. Signatory: owner. |
| `TransferRequest` | TransferWorkflow | Declares intent to transfer. Created by owner. Observer: recipient, compliance officer. |
| `ComplianceReview` | Compliance | Records compliance officer's review checks. Signatory: compliance officer. |
| `CompletedTransfer` | Settlement | Audit record of a settled transfer. Tri-party signatories. |
| `RejectedTransfer` | Settlement | Audit record of a rejected transfer. Signatory: compliance officer. |
| `TransferCancellation` | Settlement | Audit record of an owner-initiated cancellation. Signatory: owner. |
| `ComplianceDecisionLog` | Settlement | Immutable compliance decision audit entry. Signatory: compliance officer. |

## Roles

| Role | Responsibility |
|------|----------------|
| **Owner** | Holds the asset. Initiates transfer requests. Can cancel pending requests. |
| **Recipient** | Intended new holder. Observes request and review. |
| **ComplianceOfficer** | Reviews transfer requests. Approves or rejects. |

## Domain Types

- **AssetCategory**: Equity, Bond, Token, Derivative, Fund
- **Jurisdiction**: US, EU, UK, APAC
- **TransferPurpose**: Sale, Pledge, MarginCall, CorporateAction, Rebalancing, CollateralTransfer
- **ComplianceDecision**: Approved, Rejected Text

## Workflow Branches

### Branch 1: Approval (Happy Path)

```
Issuer → AssetHolding (Owner)
Owner → RequestTransfer → TransferRequest
ComplianceOfficer → StartComplianceReview → ComplianceReview
ComplianceOfficer → ApproveTransfer → New AssetHolding (Recipient) + CompletedTransfer + ComplianceDecisionLog
```

**Outcome**: Recipient receives asset. `CompletedTransfer` created with tri-party signatures. `ComplianceDecisionLog` records the approval.

### Branch 2: Rejection

```
Issuer → AssetHolding (Owner)
Owner → RequestTransfer → TransferRequest
ComplianceOfficer → StartComplianceReview → ComplianceReview
ComplianceOfficer → RejectTransfer → RejectedTransfer + ComplianceDecisionLog
```

**Outcome**: Original holding stays with owner. No settlement. `RejectedTransfer` and `ComplianceDecisionLog` created.

### Branch 3: Owner Cancellation

```
Issuer → AssetHolding (Owner)
Owner → RequestTransfer → TransferRequest
Owner → CancelTransfer → TransferCancellation
```

**Outcome**: Owner withdraws request before compliance decision. `TransferCancellation` created. Original holding remains.

## Compliance Checks

The `ComplianceReview` tracks mandatory boolean checks:

- **KYC Confirmed**: Know Your Customer verification completed
- **Transfer Limit OK**: Recipient's allowable limit not exceeded
- **Blacklist Check Passed**: Not on restricted lists
- **Sanctions Check Passed**: No sanctions matches

Optional check:
- **Regulatory Approval Obtained**: Required for certain asset types/jurisdictions

Approval requires all mandatory checks to be `True`. If any is `False`, the officer must reject.

## Audit Trail

Every transfer decision creates an immutable record:

- `CompletedTransfer`: Settlement confirmation with tri-party signatures
- `RejectedTransfer`: Rejection with reason, timestamp, and reference
- `TransferCancellation`: Owner cancellation with reason and timestamp
- `ComplianceDecisionLog`: Compliance officer's decision with full audit context

## Why DAML Fits This Use Case

DAML is well-suited for compliance-gated workflows because:

- **Explicit rights and roles**: Signatories and observers are explicit. No party can act outside their role.
- **Conditional workflows**: Choices enforce business rules. Approval requires all checks to pass.
- **Strong audit trail**: Every contract creation and exercise is recorded. Decision logs provide immutable compliance records.
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
daml script --dar .daml/dist/*.dar --script-name Scenarios:runAllScenarios
daml script --dar .daml/dist/*.dar --script-name ExampleFlows:runAllFlows
```

## Demo Files

- **Demo.daml**: Core scenarios (approval, rejection, cancellation)
- **Scenarios.daml**: Detailed scenarios with full audit verification
- **ExampleFlows.daml**: Realistic business flow examples

## Project Structure

```
digital-asset-transfer-with-compliance-hold/
├── daml.yaml
├── README.md
├── .gitattributes
└── daml/
    ├── Main.daml              # Entry point
    ├── Types.daml             # Shared type definitions
    ├── Asset.daml             # AssetHolding template and domain types
    ├── Compliance.daml        # ComplianceReview template and check types
    ├── Settlement.daml        # Audit trail templates
    ├── TransferWorkflow.daml   # TransferRequest template and workflow choices
    ├── WorkflowHelpers.daml   # Helper functions
    ├── Demo.daml              # Core demo scenarios
    ├── Scenarios.daml         # Detailed test scenarios
    └── ExampleFlows.daml      # Realistic business flow examples
```
