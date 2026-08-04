---
title: Authorization with Fraud Screening
---

# Authorization with Fraud Screening

This workflow adds a fraud risk assessment before the authorization step. A Fraud Check evaluates each transaction, and a Condition step inspects the result to decide whether to proceed with authorization or reject the transaction. This lets you block high-risk payments before they ever reach the payment provider.

## When to Use

- **Production environments** where you need to manage fraud risk.
- **High-value transactions** that warrant additional screening.
- **Merchants with a fraud provider configured** (such as a dedicated fraud scoring service).
- Any scenario where you want to **decline suspicious transactions before authorization**.

## Flow Diagram

```
                    Start Authorize
                          |
                      Fraud Check
                    /      |       \
             Completed   Paused   Requested
                 |          |         |
             Condition    Notify    Notify
            /        \
        Approved    Rejected
           |           |
       Authorize     Notify
      /   |   \   \
     C    P    R    U
     |    |    |    |
    N1   N2   N3   N4
```

Where **C** = Completed, **P** = Paused, **R** = Requested, **U** = Updated, and **N1-N4** are Notify steps.


## Step-by-Step Build Instructions

### 1. Create the Workflow

1. Navigate to **Workflow Studio** in the admin portal sidebar.
2. Click **Create Workflow**.
3. Name your workflow (for example, "Authorization with Fraud Screening").

### 2. Add the Start Authorize Trigger

1. Click **Add step** on the canvas.
2. Under **Triggers**, select **Start Authorize**.
3. The trigger appears on the canvas as the entry point for incoming authorization requests.

### 3. Add the Fraud Check Step

1. Click **Add step** and select **Fraud Check** under Actions.
2. Draw a connection line from **Start Authorize** to the **Fraud Check** step.
3. Click the **Fraud Check** step to open its settings panel.
4. Select your fraud provider from the **Provider** dropdown. The Fraud Check action supports provider selection, so you can choose which fraud service evaluates the transaction.


### 4. Handle Fraud Check Outcomes

The Fraud Check step has three outcomes: **Completed**, **Paused**, and **Requested**.

**Paused outcome:**

1. Add a **Notify** step and connect it to the **Paused** outcome.
2. Name it "Notify -- Fraud Check Paused". This fires when the fraud provider requires additional information or manual review.

**Requested outcome:**

1. Add a **Notify** step and connect it to the **Requested** outcome.
2. Name it "Notify -- Fraud Check Requested". This fires when the fraud check was submitted but the result is not yet available.

**Completed outcome** -- this requires special handling. Read on.

### 5. Add the Condition Step After Fraud Check Completed

The Fraud Check Completed outcome means the fraud provider returned a result -- but **Completed does not mean approved**. The provider may have flagged the transaction as high-risk. You need a Condition step to evaluate the result.

1. Click **Add step** and select **Condition** under Actions.
2. Draw a connection line from the **Completed** outcome of Fraud Check to the **Condition** step.
3. Click the Condition step to open its settings panel.
4. Name it "Check Fraud Result".


### 6. Configure the Condition Branches

Add a branch for approved transactions:

1. Click **Add Condition** in the settings panel.
2. Name the branch "Approved".
3. Define a rule that matches when the fraud assessment indicates the transaction is safe to proceed. Use the field dropdown to select the fraud result field, choose the appropriate operator, and enter the value that indicates approval.

The **Default** branch acts as the rejection path. If the fraud result does not match the "Approved" condition, the workflow follows the Default branch -- treating the transaction as rejected.

### 7. Handle the Rejected Path (Default Branch)

1. Add a **Notify** step and connect it to the **Default** branch of the Condition step.
2. Name it "Notify -- Fraud Rejected". This notification informs your system that the transaction was declined due to fraud risk.


### 8. Add the Authorize Step on the Approved Path

1. Click **Add step** and select **Authorize** under Actions.
2. Draw a connection line from the **Approved** branch of the Condition to the **Authorize** step.
3. Click the **Authorize** step to open its settings panel.
4. Select your payment provider from the **Provider** dropdown. The Authorize action also supports provider selection.


### 9. Add Notify Steps for Each Authorize Outcome

The Authorize step has four outcomes. Add a Notify step for each:

| Outcome | Notify Step Name |
|---|---|
| **Completed** | Notify -- Authorization Completed |
| **Paused** | Notify -- Authorization Paused |
| **Requested** | Notify -- Authorization Requested |
| **Updated** | Notify -- Authorization Updated |

Connect each outcome to its Notify step with a connection line.


### 10. Save and Activate

1. Click **Save**.
2. Review the canvas to confirm:
   - Every Fraud Check outcome has a connected step.
   - The Condition has both the Approved and Default branches connected.
   - Every Authorize outcome has a connected Notify step.
3. Activate the workflow when ready.

## Understanding the Outcome Paths

This workflow has **seven terminal paths**, each ending with a Notify step:

| Path | What Happened |
|---|---|
| Fraud Check --> Paused --> Notify | The fraud provider is waiting for additional information or manual review. |
| Fraud Check --> Requested --> Notify | The fraud check was submitted but the result is not yet available. |
| Fraud Check --> Completed --> Condition --> Default --> Notify | The fraud check completed and the transaction was rejected (did not match the Approved condition). |
| Fraud Check --> Completed --> Condition --> Approved --> Authorize --> Completed --> Notify | The transaction passed fraud screening and the authorization completed. |
| Fraud Check --> Completed --> Condition --> Approved --> Authorize --> Paused --> Notify | The transaction passed fraud screening but the authorization is waiting for an external action. |
| Fraud Check --> Completed --> Condition --> Approved --> Authorize --> Requested --> Notify | The transaction passed fraud screening and the authorization was submitted for processing. |
| Fraud Check --> Completed --> Condition --> Approved --> Authorize --> Updated --> Notify | The transaction passed fraud screening and the authorization received a status update. |

## Key Concept: Completed Does Not Mean Approved

A common mistake is to connect the Fraud Check Completed outcome directly to the Authorize step without a Condition in between. This would authorize every transaction that the fraud provider evaluated -- including the ones it flagged as high-risk.

Always add a **Condition** step after Fraud Check Completed to check the actual fraud result before deciding whether to proceed.

## Next Steps

- Add [3D Secure](workflow-studio-example-authorization-with-3ds) to the authorized path for cardholder authentication.
- Combine fraud screening and 3DS in the [Full Authorization with Fraud and 3DS](workflow-studio-example-full-authorization) workflow for risk-based routing.
- Add a **Fraud Update** step after the Authorize Completed outcome to feed the authorization result back to the fraud provider, improving future risk assessments.