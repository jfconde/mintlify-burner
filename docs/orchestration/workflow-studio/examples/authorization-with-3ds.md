---
title: Authorization with 3D Secure
---

# Authorization with 3D Secure

This workflow adds 3D Secure (3DS) cardholder authentication before the authorization step. The cardholder verifies their identity through their bank (for example, by entering a one-time password or confirming on their banking app) before the payment is authorized. A Condition step checks the 3DS result to ensure authentication succeeded before proceeding.

## When to Use

- **European payments** that require Strong Customer Authentication (SCA) under PSD2.
- **Liability shift** -- 3DS shifts fraud liability from the merchant to the card issuer for authenticated transactions.
- **Regulatory compliance** in regions that mandate cardholder authentication.
- **High-value transactions** where additional verification provides confidence.

## Flow Diagram

```
                    Start Authorize
                          |
                         3DS
                    /      |       \
             Completed   Paused   Requested
                 |          |         |
             Condition    Notify    Notify
            /        \
    Authenticated  Not Authenticated
           |              |
       Authorize        Notify
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
3. Name your workflow (for example, "Authorization with 3D Secure").

### 2. Add the Start Authorize Trigger

1. Click **Add step** on the canvas.
2. Under **Triggers**, select **Start Authorize**.
3. The trigger appears as the entry point for incoming authorization requests.

### 3. Add the 3DS Step

1. Click **Add step** and select **3DS** under Actions.
2. Draw a connection line from **Start Authorize** to the **3DS** step.
3. Click the **3DS** step to open its settings panel.
4. Select your 3DS provider from the **Provider** dropdown. The 3DS action supports provider selection, letting you choose which 3DS service handles the authentication challenge.


### 4. Handle 3DS Outcomes

The 3DS step has three outcomes: **Completed**, **Paused**, and **Requested**.

**Paused outcome:**

1. Add a **Notify** step and connect it to the **Paused** outcome.
2. Name it "Notify -- 3DS Paused". This fires when the 3DS flow is waiting for the cardholder to complete the authentication challenge (for example, entering a code or confirming on their banking app).

**Requested outcome:**

1. Add a **Notify** step and connect it to the **Requested** outcome.
2. Name it "Notify -- 3DS Requested". This fires when the 3DS challenge was initiated but is awaiting a response from the provider.

**Completed outcome** -- this requires a Condition step. Read on.

### 5. Add the Condition Step After 3DS Completed

The 3DS Completed outcome means the 3DS challenge finished -- but **Completed does not mean authenticated**. The cardholder may have failed the challenge, or the authentication may have been rejected. You need a Condition step to check the result.

1. Click **Add step** and select **Condition** under Actions.
2. Draw a connection line from the **Completed** outcome of 3DS to the **Condition** step.
3. Click the Condition step to open its settings panel.
4. Name it "Check 3DS Result".


### 6. Configure the Condition Branches

Add a branch for authenticated transactions:

1. Click **Add Condition** in the settings panel.
2. Name the branch "Authenticated".
3. Define a rule that matches when the 3DS result indicates the cardholder was successfully authenticated. Use the field dropdown to select the 3DS result field, choose the appropriate operator, and enter the value for successful authentication.

The **Default** branch handles cases where authentication was not successful. If the 3DS result does not match the "Authenticated" condition, the workflow follows the Default branch.

### 7. Handle the Not Authenticated Path (Default Branch)

1. Add a **Notify** step and connect it to the **Default** branch of the Condition step.
2. Name it "Notify -- 3DS Not Authenticated". This notification informs your system that cardholder authentication failed and the transaction should not proceed.


### 8. Add the Authorize Step on the Authenticated Path

1. Click **Add step** and select **Authorize** under Actions.
2. Draw a connection line from the **Authenticated** branch of the Condition to the **Authorize** step.
3. Click the **Authorize** step to open its settings panel.
4. Select your payment provider from the **Provider** dropdown.


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
   - Every 3DS outcome has a connected step.
   - The Condition has both the Authenticated and Default branches connected.
   - Every Authorize outcome has a connected Notify step.
3. Activate the workflow when ready.

## Understanding the Outcome Paths

This workflow has **seven terminal paths**, each ending with a Notify step:

| Path | What Happened |
|---|---|
| 3DS --> Paused --> Notify | The cardholder has been prompted to complete the 3DS challenge. The workflow pauses until they respond. |
| 3DS --> Requested --> Notify | The 3DS challenge was sent to the provider and is being processed. |
| 3DS --> Completed --> Condition --> Default --> Notify | The 3DS challenge finished but the cardholder was not authenticated. The transaction is not authorized. |
| 3DS --> Completed --> Condition --> Authenticated --> Authorize --> Completed --> Notify | The cardholder was authenticated and the authorization completed. |
| 3DS --> Completed --> Condition --> Authenticated --> Authorize --> Paused --> Notify | The cardholder was authenticated but the authorization is waiting for an external action. |
| 3DS --> Completed --> Condition --> Authenticated --> Authorize --> Requested --> Notify | The cardholder was authenticated and the authorization was submitted for processing. |
| 3DS --> Completed --> Condition --> Authenticated --> Authorize --> Updated --> Notify | The cardholder was authenticated and the authorization received a status update. |

## Key Concept: Completed Does Not Mean Authenticated

A common mistake is to connect the 3DS Completed outcome directly to the Authorize step without checking the result. This would authorize every transaction where the 3DS challenge finished -- including those where authentication failed.

Always add a **Condition** step after 3DS Completed to verify that the cardholder was actually authenticated before proceeding to Authorize.

## 3DS and Customer Experience

3D Secure adds friction to the checkout process because the cardholder must complete an additional verification step. Consider these tradeoffs:

- **Regulatory requirement:** In regions with SCA requirements (such as the European Economic Area), 3DS is mandatory for most card payments. There is no tradeoff to make -- authentication is required.
- **Liability shift:** Even where not required, 3DS shifts fraud liability from the merchant to the card issuer. This can be valuable for high-value transactions.
- **Conversion impact:** Additional verification steps can lead to cart abandonment. For lower-risk transactions, you may want to skip 3DS. See the [Full Authorization with Fraud and 3DS](workflow-studio-example-full-authorization) example for a risk-based approach.

## Next Steps

- Add [Fraud Screening](workflow-studio-example-authorization-with-fraud) before 3DS to filter out high-risk transactions before the cardholder is even prompted.
- Combine both in the [Full Authorization with Fraud and 3DS](workflow-studio-example-full-authorization) workflow for risk-based routing that only triggers 3DS when the fraud score is elevated.