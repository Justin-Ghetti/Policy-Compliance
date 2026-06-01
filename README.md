# Policy-Compliance Control Suite

> A set of automated controls that continuously check enterprise license entitlements against pricing and policy rules, create cases for any violations, and notify account owners with clear corrective steps. Built in Snowflake, Power Automate, and Salesforce.

---

## Overview

Enterprise licenses carry pricing and policy rules that govern how products can be entitled and used. Before this project, there was no systematic way to confirm that every entitlement actually complied with those rules. This suite turns that gap into a set of scheduled, automated controls: each one detects a specific class of violation in Snowflake, opens a Salesforce case, and routes a templated notification to the right account owner with instructions to fix it.

It currently covers two controls, built on a shared pattern that can be extended to additional policy checks:

- **Pricing Override Check** — flags licenses where a required pricing flag was not set correctly
- **Entitlement Policy Alignment Check** — flags entitlements that are out of alignment with the license's policy rules

---

## The Problem (Before)

Policy and pricing violations were not being caught. There was no recurring control that compared entitlements against the rules they were supposed to follow, so a misconfiguration could sit undetected indefinitely unless someone happened to notice it by chance.

That meant:

- No reliable way to know whether entitlements complied with policy at any given time
- Violations surfaced late, if at all, often only after they had already caused downstream confusion
- No audit trail showing that compliance was being actively checked

---

## The Solution (After)

Each control runs on a schedule and follows the same automated flow:

1. **Detect** — a Snowflake query evaluates entitlements against the relevant pricing or policy rule and returns only the rows that are out of compliance
2. **Create** — a Power Automate flow takes those rows and opens a pre-populated Salesforce case for each violation
3. **Notify** — the flow sends the account owner a templated email describing the specific issue and the steps to correct it

**No manual review required. Violations that used to go undetected are now caught on a recurring schedule, with a case and a clear next step every time.**

---

## Tech Stack

| Layer | Tool |
|---|---|
| Detection Logic | Snowflake SQL |
| Automation | Power Automate (scheduled) |
| Case Management | Salesforce |
| Notification | Templated HTML email via Power Automate |

---

## How It Works

> Note: rule names and thresholds below are described generically. Specific internal policy definitions are intentionally omitted.

### 1. Scheduled trigger
Each control runs on its own cadence to match how often its data changes (for example, one check runs monthly, another quarterly). Running on a schedule rather than on demand is what turns a one-time audit into an ongoing control.

### 2. Detection in SQL
The compliance logic lives entirely in the Snowflake query, not in the Power Automate flow. The query joins entitlement data against the applicable policy and pricing rules and returns only the violating rows. Everything downstream simply acts on whatever the query returns.

### 3. Case creation
Power Automate iterates the query results and creates a Salesforce case per violation, pre-populated with the account, the entitlement in question, and the nature of the issue.

### 4. Owner notification
The flow looks up the responsible account owner and sends a templated email that states the specific violation and the corrective action, so the owner can resolve it without having to investigate from scratch.

---

## Key Design Decisions

- **Detection logic lives in SQL, not in the flow.** Keeping the rules in the Snowflake query means the controls are transparent, version-able, and easy to audit or adjust. The Power Automate flow stays a thin, reusable layer that just acts on results, so adding a new control is mostly a matter of writing a new query.
- **One shared pattern across all checks.** Every control uses the same detect &rarr; case &rarr; notify structure. New policy checks plug into the existing flow rather than being rebuilt from scratch.
- **Notifications carry the fix, not just the alert.** Each email tells the owner exactly what is wrong and what to do, which is what actually drives the violation to get resolved.

---

## Impact

- Converted policy compliance from "caught by chance" to a recurring, automated control
- Surfaces violations (pricing-override gaps, out-of-alignment entitlements) that previously went undetected
- Creates a consistent audit trail: every violation produces a case and a documented notification
- Supports audit readiness and internal controls without adding manual review work
- Extensible: additional policy checks can be added on the same pattern

---

## Notes on Sensitive Data

This repository is a sanitized portfolio version. Schema names, table names, specific policy and pricing rule definitions, and account identifiers have been omitted or replaced with generic descriptions. No real customer or internal system data is included.
