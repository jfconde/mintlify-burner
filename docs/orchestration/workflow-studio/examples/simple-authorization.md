---
title: Simple Authorization
---

# Simple Authorization

This is the simplest possible authorization workflow. It receives an authorization request, sends it to a payment provider, and notifies your system of the result on every outcome path.

## When to Use

- **Single-provider setups** where all authorizations go to the same provider.
- **Testing and development** when you want a minimal workflow to verify your integration.
- **Simple payment flows** that do not require fraud screening, 3D Secure, or conditional routing.

## Flow Diagram

```
                Start Authorize
                      |
                  Authorize
              /    |      \      \
         Completed Paused Requested Updated
             |       |       |        |
          Notify   Notify  Notify   Notify
```

The workflow has one trigger, one action, and four Notify steps -- one for each possible Authorize outcome.


## Step-by-Step Build Instructions

### 1. Create the Workflow

1. Navigate to **Workflow Studio** in the admin portal sidebar.
2. Click **Create Workflow**.
3. Name your workflow (for example, "Simple Authorization").

### 2. Add the Start Authorize Trigger

1. Click the **Add step** button on the canvas.
2. Under **Triggers**, select **Start Authorize**.
3. The trigger appears on the canvas. This is the entry point -- it fires whenever your system sends an authorization request.


### 3. Add the Authorize Step

1. Click **Add step** again.
2. Under **Actions**, select **Authorize**.
3. Draw a connection line from **Start Authorize** to the **Authorize** step.
4. Click the **Authorize** step to open its settings panel.
5. Select your payment provider from the **Provider** dropdown.


### 4. Add Notify Steps for Each Outcome

The Authorize step has four outcomes: **Completed**, **Paused**, **Requested**, and **Updated**. Each one needs a Notify step.

**Completed outcome:**

1. Click **Add step** and select **Notify**.
2. Draw a connection line from the **Completed** outcome to the Notify step.
3. Click the Notify step to open its settings panel.
4. Name it "Notify -- Authorization Completed".

**Paused outcome:**

1. Add another **Notify** step.
2. Connect it to the **Paused** outcome.
3. Name it "Notify -- Authorization Paused".

**Requested outcome:**

1. Add another **Notify** step.
2. Connect it to the **Requested** outcome.
3. Name it "Notify -- Authorization Requested".

**Updated outcome:**

1. Add another **Notify** step.
2. Connect it to the **Updated** outcome.
3. Name it "Notify -- Authorization Updated".


### 5. Save and Activate

1. Click **Save** to save the workflow.
2. Review the canvas to confirm every outcome has a connected Notify step.
3. Activate the workflow when you are ready to go live.

## Understanding the Outcomes

| Outcome | What It Means | Your System Receives |
|---|---|---|
| **Completed** | The authorization finished. The provider returned a final result (approved or declined). | A notification with the authorization result so you can proceed with the order or inform the customer. |
| **Paused** | The authorization is waiting for an external action, such as a customer completing a redirect or a bank verification. | A notification that the authorization is pending. The workflow resumes automatically when the external action completes. |
| **Requested** | The authorization was submitted to the provider but the result is not yet available. The provider is processing it asynchronously. | A notification that the authorization is in progress. The final result arrives through a separate event-driven workflow. |
| **Updated** | The provider sent a status update after the initial authorization response. This can happen when a provider corrects a previous result. | A notification with the updated status so your system can reconcile. |

## What This Workflow Does Not Include

This workflow sends every authorization to a single provider without any pre-processing. It does not include:

- **Fraud screening** -- Transactions are not checked for fraud risk before authorization. See [Authorization with Fraud Screening](workflow-studio-example-authorization-with-fraud) to add this.
- **3D Secure authentication** -- The cardholder is not prompted for additional verification. See [Authorization with 3D Secure](workflow-studio-example-authorization-with-3ds) to add this.
- **Conditional routing** -- All transactions follow the same path regardless of amount, currency, or card type. Add a Condition step before Authorize to route transactions to different providers.

## Next Steps

Once this workflow is running, consider enhancing it:

- Add a [Fraud Check](workflow-studio-example-authorization-with-fraud) before the Authorize step to screen transactions for risk.
- Add [3D Secure](workflow-studio-example-authorization-with-3ds) for cardholder authentication and SCA compliance.
- Combine both in the [Full Authorization with Fraud and 3DS](workflow-studio-example-full-authorization) workflow for comprehensive security.