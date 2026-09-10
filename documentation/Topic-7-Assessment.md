# Topic 7 Assessment — Technical Documentation

## Objective

Build a self-correcting AI Agent in n8n that audits transaction records, invokes a validation tool, performs iterative self-correction, and produces standardized JSON output.

## Implementation

The workflow is named **Self-Correcting Agentic Auditor**.

### Components

- Manual Trigger: `Run Audit`
- Code node: `Mock Transaction Data`
- Split In Batches: `Process One At A Time`
- AI Agent: `Transaction Auditor Agent`
- Google Gemini Chat Model: temperature `0.0`, max output tokens `2048`
- Code Tool: `validate_transaction`
- Structured Output Parser with `additionalProperties: false`
- IF router: `Validation Router`
- `Valid Transaction Output`
- `Invalid Transaction Output`
- Wait node: `Rate Limit Pacing` at 20 seconds

## Agent Behavior

The agent must call the validator before declaring validity. When validation fails, it examines the errors, applies only safe and minimal corrections, and calls the validator again. The maximum configured agent iterations is 4.

Safe correction examples:

- Currency normalization: `usd` → `USD`
- Status normalization: `done` → `completed`
- Unambiguous date formatting: `09/06/2026` → `2026-09-06`

The agent must not fabricate missing customer IDs or amounts, guess values for invalid amounts, or map an unknown transaction type to a valid type.

## Validator

The validator is deterministic JavaScript and uses no LLM or external network. It checks required fields, positive amount, currency enum, transaction type enum, status enum, and date format.

## Structured Output

The final schema contains:

- `transaction_id`
- `status`
- `original_transaction`
- `corrected_transaction`
- `validation_errors`
- `corrections_made`
- `iterations`
- `validation_passed`

## Test Dataset

Eight deterministic transactions are configured, covering valid, correctable, and uncorrectable scenarios.

## Live Evidence

Execution #148 successfully demonstrated real Gemini calls and real validator tool calls:

- TXN-003: `usd` → `USD`, VALID, 2 iterations.
- TXN-005: `done` → `completed`, VALID, 2 iterations.
- TXN-008: missing amount + invalid transaction type, no unsafe correction, INVALID, 1 iteration.

## Conditional Routing

The router evaluates the structured `status` field. `VALID` goes to the valid output branch; all other results go to the invalid/exception branch.

## Quota Resilience

The workflow processes one transaction at a time and waits 20 seconds between transactions. This pacing was introduced to avoid the available Gemini free-tier request-rate limitation.

## Assessment Evidence Note

The complete eight-case dataset is configured in the exported workflow. Only the three representative cases listed above are claimed as live-tested in Execution #148.
