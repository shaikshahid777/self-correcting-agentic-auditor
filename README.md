<!-- SHOWCASE_START --><div align="center">[![Typing](https://readme-typing-svg.demolab.com?font=Fira+Code&weight=700&size=22&duration=2800&pause=900&color=58A6FF&center=true&vCenter=true&width=900&lines=self%20correcting%20agentic%20auditor;AI%20%7C%20Automation%20%7C%20Engineering;Explore%20the%20project%20%F0%9F%9A%80)](https://github.com/shaikshahid777/self-correcting-agentic-auditor)<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0D1117,50:161B22,100:58A6FF&height=110&section=header&text=self-correcting-agentic-auditor&fontSize=26&fontColor=FFFFFF&animation=twinkling&fontAlignY=65" width="100%" alt="Animated project banner"/>

[![Repository](https://img.shields.io/badge/Repository-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/shaikshahid777/self-correcting-agentic-auditor) [![Issues](https://img.shields.io/badge/Report-Issue-red?style=for-the-badge&logo=github)](https://github.com/shaikshahid777/self-correcting-agentic-auditor/issues/new) [![Stars](https://img.shields.io/github/stars/shaikshahid777/self-correcting-agentic-auditor?style=for-the-badge&logo=github)](https://github.com/shaikshahid777/self-correcting-agentic-auditor/stargazers) [![Fork](https://img.shields.io/github/forks/shaikshahid777/self-correcting-agentic-auditor?style=for-the-badge&logo=github)](https://github.com/shaikshahid777/self-correcting-agentic-auditor/fork) [![Profile](https://img.shields.io/badge/Profile-Visit-0A66C2?style=for-the-badge&logo=github)](https://github.com/shaikshahid777)</div>

> ✨ **Project Showcase Mode:** animated banner • interactive navigation • live repository actions

[🚀 Repository](https://github.com/shaikshahid777/self-correcting-agentic-auditor) · [🐞 Report Issue](https://github.com/shaikshahid777/self-correcting-agentic-auditor/issues/new) · [⭐ Star](https://github.com/shaikshahid777/self-correcting-agentic-auditor/stargazers) · [🔱 Fork](https://github.com/shaikshahid777/self-correcting-agentic-auditor/fork) · [👤 Profile](https://github.com/shaikshahid777)

<!-- SHOWCASE_END -->

# Self-Correcting Agentic Auditor

## Topic 7 – Advanced Planning, Reasoning & Self-Correction

An n8n-based self-correcting transaction auditor that combines deterministic validation with Gemini-powered agentic reasoning.

## Overview

The workflow receives deterministic mock transaction records, validates them through a non-LLM Code Tool, asks a Google Gemini-powered AI Agent to reason about safe corrections, re-validates corrected transactions, produces strict structured JSON, and routes each transaction to a VALID or INVALID output branch.

## Workflow

```text
Run Audit
  ↓
Mock Transaction Data
  ↓
Process One At A Time
  ↓
Transaction Auditor Agent
  ├── Google Gemini Chat Model
  ├── validate_transaction Code Tool
  └── Structured Output Parser
  ↓
Validation Router
  ├── VALID → Valid Transaction Output
  └── INVALID → Invalid Transaction Output
  ↓
Rate Limit Pacing → next transaction
```

## Key Configuration

- **LLM:** Google Gemini
- **Temperature:** 0.0
- **Max Output Tokens:** 2048
- **AI Agent Max Iterations:** 4
- **Validator:** deterministic Code Tool; no LLM
- **Structured Output Parser:** strict schema with `additionalProperties: false`
- **Processing:** one transaction at a time
- **Pacing:** 20-second wait between transactions to respect the available Gemini free-tier quota

## Validation Rules

The validator checks:

- Required fields: `transaction_id`, `customer_id`, `amount`, `currency`, `transaction_date`, `transaction_type`, `status`
- Amount greater than zero
- Currency in `USD`, `EUR`, `GBP`, `INR`, `JPY`
- Transaction type in `purchase`, `refund`, `transfer`, `withdrawal`, `deposit`
- Status in `pending`, `completed`, `failed`, `cancelled`
- Transaction date in `YYYY-MM-DD` format

## Self-Correction Behavior

The agent must validate using the tool before claiming success. When validation fails, it inspects the tool errors, applies only safe minimal corrections, and re-validates.

Safe corrections demonstrated include:

- `usd` → `USD`
- `done` → `completed`
- Unambiguous date reformatting such as `09/06/2026` → `2026-09-06`

Unsafe corrections are prohibited. The agent must not invent missing customer IDs or amounts, guess a negative amount, or map an unknown transaction type to a valid type.

## Mock Test Dataset

The workflow contains eight deterministic cases:

| Transaction | Case | Expected outcome |
|---|---|---|
| TXN-001 | Fully valid | VALID immediately |
| TXN-002 | Missing `customer_id` | INVALID / uncorrectable |
| TXN-003 | `currency = usd` | VALID after correction |
| TXN-004 | Negative amount | INVALID / uncorrectable |
| TXN-005 | `status = done` | VALID after correction |
| TXN-006 | Date `09/06/2026` | VALID after reformat |
| TXN-007 | `eur` + `Completed` casing | VALID after correction |
| TXN-008 | Missing amount + `quantum_teleport` type | INVALID / uncorrectable |

## Live Validation Evidence

A controlled live run, **Execution #148**, was completed with the real Gemini credential.

- **TXN-003:** `usd` → `USD`; re-validation passed; **VALID**, 2 iterations.
- **TXN-005:** `done` → `completed`; re-validation passed; **VALID**, 2 iterations.
- **TXN-008:** missing amount + invalid transaction type; no unsafe correction; **INVALID**, 1 iteration.

The remaining five cases are present in the full deterministic dataset but were not claimed as live-tested in Execution #148.

## Deliverables

- `workflow/Self-Correcting Agentic Auditor.json` — exported n8n workflow
- `documentation/Topic-7-Assessment.pdf` — assessment documentation
- `validation-results/Execution-148.md` — live validation evidence

## Demo

Loom: https://www.loom.com/share/f099a3e01bd44d569f1b0b4fcb9b1e2e

## Notes

The workflow is saved in n8n and the exported JSON preserves the Gemini model configuration, validator, structured parser, self-correction instructions, conditional routing, sequential processing, and rate-limit pacing.
