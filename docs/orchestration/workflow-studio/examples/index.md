---
title: Workflow Examples
---

# Workflow Examples

This section provides ready-to-use workflow configurations for common payment scenarios. Each example walks you through a complete workflow from trigger to terminal Notify steps, with step-by-step build instructions and visual diagrams.

All examples follow these best practices:

- **Notify on every terminal path.** Every branch of the workflow ends with a Notify step so your system is always informed of the outcome.
- **All outcomes handled.** Every outcome on every action step has a connection line leading to a next step. No outcomes are left unconnected.
- **Conditions evaluate results.** When an action completes, a Condition step checks the result before proceeding -- because Completed does not always mean approved.

## Example Workflows

| Example | Description | Key Steps |
|---|---|---|
| [Simple Authorization](workflow-studio-example-simple-authorization) | The most basic authorization workflow. A single Authorize step with Notify on every outcome. | Start Authorize, Authorize, Notify |
| [Authorization with Fraud Screening](workflow-studio-example-authorization-with-fraud) | Adds a Fraud Check before authorization to screen transactions for risk. Uses a Condition to evaluate the fraud result and reject high-risk transactions. | Start Authorize, Fraud Check, Condition, Authorize, Notify |
| [Authorization with 3D Secure](workflow-studio-example-authorization-with-3ds) | Adds 3D Secure cardholder authentication before authorization for PSD2/SCA compliance and liability shift. | Start Authorize, 3DS, Condition, Authorize, Notify |
| [Full Authorization with Fraud and 3DS](workflow-studio-example-full-authorization) | The most comprehensive authorization flow. Combines fraud screening with risk-based 3DS routing to balance security and customer experience. | Start Authorize, Fraud Check, Condition, 3DS, Authorize, Notify |

## How to Use These Examples

Each example includes:

1. **A flow diagram** showing the complete workflow from trigger to terminal steps.
2. **Step-by-step build instructions** you can follow in Workflow Studio.
3. **Outcome explanations** describing what each path means and why it matters.
4. **When to use** guidance to help you choose the right workflow for your situation.

You can build these workflows exactly as described, or use them as starting points and customize them to fit your specific requirements -- for example, by adding extra Condition steps to route to different providers based on currency or payment method.