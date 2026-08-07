---
title: Actions
---

# Actions

Actions are the building blocks of every workflow. Each action performs a specific payment operation -- authorizing a transaction, capturing funds, running a fraud check, and more. You place actions on the canvas after a trigger or after the outcome of a previous step, and connect them together to build your payment flow.


## How actions work

When the workflow reaches an action step, the platform executes the operation and produces an **outcome**. Each outcome represents a different result of the action and appears as a separate connection point on the step. You connect the next step in your flow to the appropriate outcome, so the workflow knows where to go based on what happens.

For example, an Authorize step has four outcomes: Completed, Paused, Requested, and Updated. You can connect a Notify step to the Completed outcome to inform your integration of a successful authorization, and connect a different Notify step to the Paused outcome to handle cases where the payment requires additional interaction.


## Outcomes

Most actions produce one or more of the following outcomes:

| Outcome | Meaning |
|---------|---------|
| **Completed** | The action finished successfully. The operation has been processed and a final result is available. |
| **Paused** | The action is waiting for an external response before it can continue. This typically happens when the payment requires additional customer interaction, such as a redirect or a challenge. |
| **Requested** | The action has been submitted to the provider and is pending processing. The provider has acknowledged the request but has not yet returned a final result. |
| **Updated** | The action received a status update from the provider after the initial response. This outcome is available on Authorize, Capture, Cancel, and Refund actions. |

The **Notify** action is special -- it has no outcomes because it is always a terminal step. It sends a notification and the workflow path ends there.

## Provider selection

Some actions support **provider selection**, which lets you choose which payment provider configuration handles the operation. When you select one of these actions and open its settings panel, you will see a provider dropdown where you can pick from the provider configurations available for your merchant.

Actions that support provider selection:

- **Authorize** -- Choose which payment provider processes the authorization.
- **Fraud Check** -- Choose which fraud provider performs the risk assessment.
- **Fraud Update** -- Choose which fraud provider receives the transaction outcome update.
- **3DS** -- Choose which 3D Secure provider performs the authentication check.
- **Provision Network Token** -- Choose which provider provisions the network token.

All other actions use the provider that is already associated with the payment context and do not require you to select one.


## All available actions

### Authorize

Sends an authorization request to a payment provider. Authorization verifies the payment method and reserves the requested amount without moving funds. Supports **provider selection**.

**Outcomes:** Completed, Paused, Requested, Updated

Use a Condition step after Completed to check whether the authorization succeeded. Pair with a Fraud Update step after successful authorization to improve future fraud scoring.


### Capture

Captures (settles) a previously authorized payment, moving the reserved funds from the customer to the merchant. Typically placed in workflows triggered by Start Capture.

**Outcomes:** Completed, Paused, Requested, Updated

> **Tip:** For auto-capture scenarios, you can use a Handle Payment Status Update trigger to automatically capture payments when authorization completes asynchronously.


### Cancel

Cancels (voids) a payment, releasing the authorization hold on the customer's funds. Cancellation workflows should be straightforward since time-sensitivity matters -- the sooner a cancellation is processed, the sooner the hold is released.

**Outcomes:** Completed, Paused, Requested, Updated


### Refund

Returns funds to the customer for a previously captured payment. Use a Condition step before the Refund action if you need to handle full and partial refunds differently.

**Outcomes:** Completed, Paused, Requested, Updated


### Fraud Check

Performs a fraud risk assessment by sending transaction details to a fraud detection provider. Supports **provider selection**. Most commonly placed before the Authorize step to screen transactions for risk.

**Outcomes:** Completed, Paused, Requested

After Completed, add a Condition step to evaluate the fraud result. Available fields in the condition include the fraud decision (approve, review, decline) and fraud score. Route approved transactions to Authorize and declined transactions to Notify.


### Fraud Update

Sends the transaction outcome to a fraud provider so it can update its risk models. Supports **provider selection**. Typically placed after an Authorize, Capture, or Refund step completes -- on both success and failure paths -- so the fraud provider has the full picture.

**Outcomes:** Completed, Paused, Requested


### Payout

Initiates a payout to send funds to a recipient. This action is used in payout-specific workflows.

**Outcomes:** Completed, Paused, Requested


### 3D Secure (3DS)

Performs cardholder authentication by initiating a 3D Secure verification flow. The cardholder is prompted to verify their identity through their bank's authentication process (e.g., entering a one-time password or confirming on their banking app). Supports **provider selection**.

**Outcomes:** Completed, Paused, Requested

After Completed, add a Condition step to check whether authentication succeeded before proceeding to Authorize. The Completed outcome means the 3DS challenge finished, but not necessarily that authentication was successful.

Common patterns:

- **3DS before Authorize:** Start Authorize --> 3DS --> Condition --> Authorize
- **3DS on Authorize pause:** Authorize --> Paused --> 3DS --> Condition --> Authorize (retry)
- **Risk-based 3DS:** Fraud Check --> Condition --> 3DS (medium risk) or Authorize (low risk)

> **Tip:** In addition to the standalone 3DS step, you can control 3DS behavior directly on the Authorize step using 3DS mode settings (Default, Force, or Skip). The standalone step gives you more control over the flow.


### Notify

Sends a webhook notification to your system with the workflow execution result. Notify is a **terminal step** -- it has no outcomes and no outgoing connection lines. The workflow branch ends after Notify.

**Outcomes:** None (terminal step)

The notification destination, authentication, and payload format are managed in the **Notifications Configuration** section of your portal settings. The Notify step triggers the notification; your notification configuration determines where and how it is delivered.

> **Best practice:** Place a Notify step at the end of **every** terminal path in your workflow -- both success and failure paths. Name your Notify steps clearly (e.g., "Notify -- Authorization Failed") so they are easy to identify on the canvas and in execution logs.


### Create Instrument

Creates or retrieves a stored payment instrument (tokenized payment method) for the customer. The resulting instrument data is automatically available to downstream steps. Typically placed before Authorize in flows that use stored payment methods.

**Outcomes:** Completed, Paused, Requested

> **Tip:** Not all payment methods support instruments. Use a Condition step before Create Instrument to check support, and provide an alternative path for unsupported methods.


### Provision Network Token

Requests a network token from a card network (Visa, Mastercard, etc.) for a stored payment instrument. Network tokens replace the actual card number, improving authorization rates and security. Supports **provider selection**.

**Outcomes:** Completed, Paused, Requested

Typically placed after Create Instrument and before Authorize. Use a Condition after Completed to check the result, and fall back to authorizing without a token if provisioning fails.

> **Tip:** Network tokens are for card payments only. Route non-card payments around this step using a Condition.


### Lookup

Looks up available payment options. Typically used in checkout initialization flows to determine which payment methods are available for the current transaction context.

**Outcomes:** Completed, Paused, Requested


### Condition

Evaluates a set of rules and branches the workflow into different paths based on the results. This is the primary way to add decision logic to your workflows. For details on configuring conditions, see the [Conditions](./conditions) section.

**Outcomes:** Custom condition branches + Default branch

Branches are evaluated top to bottom -- the first match wins. The Default branch catches anything your custom conditions do not cover. Always connect the Default branch to a Notify step at minimum.



## Summary

| Action | Outcomes | Provider Selection |
|--------|----------|-------------------|
| Authorize | Completed, Paused, Requested, Updated | Yes |
| Capture | Completed, Paused, Requested, Updated | No |
| Cancel | Completed, Paused, Requested, Updated | No |
| Refund | Completed, Paused, Requested, Updated | No |
| Fraud Check | Completed, Paused, Requested | Yes |
| Fraud Update | Completed, Paused, Requested | Yes |
| Payout | Completed, Paused, Requested | No |
| 3DS | Completed, Paused, Requested | Yes |
| Notify | None (terminal step) | No |
| Create Instrument | Completed, Paused, Requested | No |
| Provision Network Token | Completed, Paused, Requested | Yes |
| Lookup | Completed, Paused, Requested | No |
| Condition | Custom branches + Default | No |


## Adding an action to a workflow

1. Open your workflow in Workflow Studio.
2. In the step palette on the left side of the canvas, find the action you want to add.
3. Drag the action onto the canvas, or click it to add it.
4. Draw a connection line from the outcome of a previous step (or from the trigger) to the new action.
5. Click the action step on the canvas to open its settings panel and configure any required settings.


## Best practices

- **Connect every outcome.** Each outcome on an action step should have a connection line leading to the next step. If an outcome is left unconnected, the workflow will end silently at that point without notifying your integration.
- **Add a Notify step on every terminal path.** Whether the workflow succeeds or encounters an issue, the final step on each path should be a Notify action. This ensures your integration is always informed of the workflow result.
- **Use conditions after outcomes when needed.** If you need different behavior depending on details of the outcome (for example, routing based on a response code), add a condition step after the outcome before the next action.
- **Review provider selection carefully.** For actions that support provider selection, make sure the correct provider configuration is selected in the settings panel before activating the workflow.