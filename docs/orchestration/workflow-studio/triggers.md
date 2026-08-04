---
title: Triggers
---
# Triggers

Every workflow begins with a **trigger**. The trigger defines what event or API call starts the workflow and determines the data that is available to all subsequent steps. Each workflow has at least one trigger, and it is always the first step on the canvas.

## API triggers vs. event triggers

Workflow Studio offers two kinds of triggers:

| Kind              | How it activates                                                     | When to use                                                                                                        |
| ----------------- | -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **API trigger**   | Activated when a specific API endpoint is called by your integration | Payment operations initiated by your system, such as authorizing, capturing, or refunding a payment                |
| **Event trigger** | Activated automatically when a system event occurs                   | Asynchronous updates that arrive from external sources, such as dispute status changes or payment status callbacks |

### API triggers

API triggers start a workflow in response to an incoming API request. Some are invoked through the client API (called from your front-end integration) while others are invoked through the merchant API (called from your back-end).

### Event triggers

Event triggers start a workflow automatically when the platform detects a specific event. You do not need to call an API endpoint to activate them -- they fire as soon as the matching event is received.

## Available triggers

### Start Authorize

**Type:** API (client)

Activates when the `/authorize` endpoint is called. This is the most common trigger and serves as the entry point for payment authorization flows. The authorization request data -- including payment amount, currency, payment method details, and shopper information -- becomes available to all subsequent steps.

Common patterns:

* **Basic authorization:** Start Authorize --> Authorize --> Notify on each outcome
* **With fraud screening:** Start Authorize --> Fraud Check --> Condition --> Authorize or Notify
* **With 3DS:** Start Authorize --> 3DS --> Authorize --> Notify on each outcome
* **Full flow:** Start Authorize --> Fraud Check --> Condition --> 3DS --> Authorize --> Notify

### Start Capture

**Type:** API (merchant)

Activates when the `/capture` endpoint is called to settle a previously authorized payment. The capture request data -- including the payment reference and capture amount -- becomes available to all subsequent steps.

Common patterns:

* **Basic capture:** Start Capture --> Capture --> Notify on each outcome
* **With fraud update:** Start Capture --> Capture --> Fraud Update --> Notify

### Start Cancel

**Type:** API (merchant)

Activates when the `/cancel` endpoint is called to void a payment. Cancellation workflows tend to be straightforward since the goal is to release the hold on the customer's funds as quickly as possible.

Common patterns:

* **Basic cancellation:** Start Cancel --> Cancel --> Notify on each outcome
* **Conditional cancellation:** Start Cancel --> Condition (route by method or provider) --> Cancel --> Notify

### Start Refund

**Type:** API (merchant)

Activates when the `/refund` endpoint is called to return funds to the customer. The refund request data -- including the payment reference and refund amount -- becomes available to all subsequent steps.

Common patterns:

* **Basic refund:** Start Refund --> Refund --> Notify on each outcome
* **Conditional refund:** Start Refund --> Condition (full vs. partial) --> Refund --> Notify
* **With fraud update:** Start Refund --> Refund --> Fraud Update --> Notify

### Start Order Update

**Type:** API (merchant)

Activates when the `/order-update` endpoint is called to update order details on an existing payment. This trigger is used when order information changes after the initial authorization -- for example, when shipping details or line items are updated.

Common patterns:

* **Basic update:** Start Order Update --> Authorize --> Notify on each outcome
* **Conditional update:** Start Order Update --> Condition (update type) --> Authorize --> Notify

> **Note:** Not all providers support order updates. Use a Condition step if you need to handle providers with different levels of support.

### Start Lookup

**Type:** API (merchant)

Activates when the `/lookup` endpoint is called to look up available payment options. Lookup workflows are typically lightweight and fast since they are called during checkout initialization.

Common patterns:

* **Basic lookup:** Start Lookup --> Lookup --> Notify on each outcome
* **Conditional lookup:** Start Lookup --> Condition (currency or country) --> Lookup --> Notify

### Handle Dispute Update

**Type:** Event

Activates automatically when a **DisputeUpdated** event is received -- for example, when a new chargeback is opened, a dispute status changes, or a chargeback is won or lost. No API call from your integration is required.

Common patterns:

* **Basic notification:** Handle Dispute Update --> Notify
* **Conditional handling:** Handle Dispute Update --> Condition (dispute status) --> Notify (per status)

> **Tip:** Dispute workflows are event-driven and can fire at any time. Design your workflow to handle updates gracefully, even if they arrive out of order or contain unexpected statuses.

### Handle Payment Status Update

**Type:** Event

Activates automatically when a payment status changes asynchronously -- for example, when a provider reports a delayed authorization result, a capture confirmation, or a status correction. This is especially relevant for payment methods with asynchronous processing, such as bank transfers.

Common patterns:

* **Basic notification:** Handle Payment Status Update --> Notify
* **Conditional handling:** Handle Payment Status Update --> Condition (status) --> Notify or Capture
* **Auto-capture:** Handle Payment Status Update --> Condition (authorized?) --> Capture --> Notify

> **Tip:** Design your workflow to handle duplicate or unexpected status updates gracefully. A payment may receive multiple status updates as it moves through processing.

## Summary

| Trigger                      | Type           | Description                                                                  |
| ---------------------------- | -------------- | ---------------------------------------------------------------------------- |
| Start Authorize              | API (client)   | Starts a payment authorization flow when the `/authorize` endpoint is called |
| Start Capture                | API (merchant) | Starts a capture flow to settle a previously authorized payment              |
| Start Cancel                 | API (merchant) | Starts a cancellation flow to void a payment                                 |
| Start Refund                 | API (merchant) | Starts a refund flow to return funds to the customer                         |
| Start Order Update           | API (merchant) | Starts a flow to update order details on an existing payment                 |
| Start Lookup                 | API (merchant) | Starts a flow to look up available payment options                           |
| Handle Dispute Update        | Event          | Starts a flow when a dispute status changes                                  |
| Handle Payment Status Update | Event          | Starts a flow when a payment status changes asynchronously                   |

## Adding a trigger to a workflow

When you create a new workflow, you choose its trigger as part of the creation process:

1. Open **Workflow Studio** from the main navigation.
2. Click **Create workflow**.
3. In the creation dialog, select the trigger that matches the flow you want to build.
4. Give the workflow a name and click **Create**.

The trigger step appears as the first step on the canvas. From here, you can add action steps, condition steps, and Notify steps to build out the rest of your workflow.

> **Tip:** Every terminal path in your workflow should end with a **Notify** step. This ensures that your integration always receives a response, whether the workflow succeeds or encounters an error.

## What happens next

After your trigger fires, the data from the incoming request or event becomes available to every subsequent step. You can use this data to:

* Configure action steps (for example, selecting a provider or setting amounts).
* Define conditions that branch the workflow based on request properties.
* Include relevant details in Notify steps sent back to your integration.

For details on the actions you can add after a trigger, see the [Actions](workflow-studio-actions) section.