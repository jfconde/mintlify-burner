---
title: Full Authorization with Fraud and 3DS
---

# Full Authorization with Fraud and 3DS

This is the most comprehensive authorization workflow. It combines fraud screening with risk-based 3D Secure routing so that low-risk transactions are authorized quickly, medium-risk transactions go through 3DS for additional verification, and high-risk transactions are rejected outright. This approach balances security with customer experience -- only adding friction when the risk warrants it.

## When to Use

- **Production environments** with both a fraud provider and a 3DS provider configured.
- **Risk-based authentication** where you want to apply 3DS selectively based on fraud score rather than on every transaction.
- **European payments** that must comply with PSD2/SCA but where you want to minimize unnecessary 3DS challenges for low-risk transactions.
- **Merchants processing a mix of transaction risk levels** who need different handling for different risk tiers.

## Flow Diagram

```
                         Start Authorize
                               |
                          Fraud Check
                       /       |       \
                Completed    Paused   Requested
                    |          |         |
                Condition    Notify    Notify
              /     |     \
         Low Risk  Med Risk  High Risk (Default)
            |         |            |
        Authorize    3DS         Notify
       / | \  \     / |  \
      C  P  R  U  Co  Pa  Rq
      |  |  |  |   |   |   |
     N1 N2 N3 N4  Cond N5  N6
                  / \
            Auth   Default
           / | \ \    |
          C  P  R  U  N7
          |  |  |  |
         N8 N9 N10 N11
```

Where **C/Co** = Completed, **P/Pa** = Paused, **R/Rq** = Requested, **U** = Updated, **Cond** = Condition, **Auth** = Authorize, and **N1-N11** are Notify steps.

## Step-by-Step Build Instructions

### 1. Create the Workflow

1. Navigate to **Workflow Studio** in the admin portal sidebar.
2. Click **Create Workflow**.
3. Name your workflow (for example, "Full Authorization with Fraud and 3DS").

### 2. Add the Start Authorize Trigger

1. Click **Add step** on the canvas.
2. Under **Triggers**, select **Start Authorize**.
3. The trigger appears on the canvas as the entry point.

### 3. Add the Fraud Check Step

1. Click **Add step** and select **Fraud Check** under Actions.
2. Draw a connection line from **Start Authorize** to the **Fraud Check** step.
3. Click the **Fraud Check** step to open its settings panel.
4. Select your fraud provider from the **Provider** dropdown.

### 4. Handle Fraud Check Non-Completed Outcomes

**Paused outcome:**

1. Add a **Notify** step and connect it to the **Paused** outcome.
2. Name it "Notify -- Fraud Check Paused".

**Requested outcome:**

1. Add a **Notify** step and connect it to the **Requested** outcome.
2. Name it "Notify -- Fraud Check Requested".

### 5. Add the Risk-Level Condition

This is the key decision point of the workflow. The Condition step evaluates the fraud result and routes the transaction into one of three risk tiers.

1. Click **Add step** and select **Condition** under Actions.
2. Draw a connection line from the **Completed** outcome of Fraud Check to the **Condition** step.
3. Click the Condition step to open its settings panel.
4. Name it "Check Risk Level".

### 6. Configure the Risk-Level Branches

Add two custom branches. The Default branch serves as the high-risk path.

**Low Risk branch:**

1. Click **Add Condition**.
2. Name the branch "Low Risk".
3. Define a rule that matches when the fraud assessment indicates low risk. Use the field dropdown to select the fraud result field, choose the appropriate operator, and set the threshold for low-risk transactions.

**Medium Risk branch:**

1. Click **Add Condition** again.
2. Name the branch "Medium Risk".
3. Define a rule that matches when the fraud assessment indicates medium risk -- above the low-risk threshold but below the level you consider unacceptable.

**Default branch (High Risk):**

The Default branch catches any transaction that does not match the Low Risk or Medium Risk conditions. These are the highest-risk transactions that should be rejected.

> **Tip:** Make sure the Low Risk branch is ordered above Medium Risk in the settings panel. Branches are evaluated top to bottom, so the most specific condition should come first.

### 7. Build the High-Risk Path (Default Branch)

High-risk transactions are rejected immediately without reaching a payment provider.

1. Add a **Notify** step and connect it to the **Default** branch.
2. Name it "Notify -- Fraud Rejected (High Risk)".

### 8. Build the Low-Risk Path (Direct Authorization)

Low-risk transactions skip 3DS and go directly to authorization for the fastest checkout experience.

1. Click **Add step** and select **Authorize** under Actions.
2. Draw a connection line from the **Low Risk** branch to the **Authorize** step.
3. Click the **Authorize** step to open its settings panel.
4. Select your payment provider from the **Provider** dropdown.
5. Name the step "Authorize (Low Risk)" to distinguish it on the canvas.

Add Notify steps for all four Authorize outcomes:

| Outcome       | Notify Step Name                           |
| ------------- | ------------------------------------------ |
| **Completed** | Notify -- Low Risk Authorization Completed |
| **Paused**    | Notify -- Low Risk Authorization Paused    |
| **Requested** | Notify -- Low Risk Authorization Requested |
| **Updated**   | Notify -- Low Risk Authorization Updated   |

Connect each outcome to its Notify step.

### 9. Build the Medium-Risk Path (3DS Then Authorization)

Medium-risk transactions require 3D Secure authentication before authorization. This adds cardholder verification for elevated-risk transactions while keeping the checkout frictionless for low-risk ones.

**Add the 3DS step:**

1. Click **Add step** and select **3DS** under Actions.
2. Draw a connection line from the **Medium Risk** branch to the **3DS** step.
3. Click the **3DS** step to open its settings panel.
4. Select your 3DS provider from the **Provider** dropdown.

**Handle 3DS non-completed outcomes:**

1. Add a **Notify** step and connect it to the **Paused** outcome. Name it "Notify -- 3DS Paused".
2. Add a **Notify** step and connect it to the **Requested** outcome. Name it "Notify -- 3DS Requested".

**Add a Condition after 3DS Completed:**

1. Click **Add step** and select **Condition** under Actions.
2. Draw a connection line from the **Completed** outcome of 3DS to the Condition step.
3. Name it "Check 3DS Result".
4. Add a branch named "Authenticated" with a rule that matches successful 3DS authentication.
5. The **Default** branch handles failed authentication.

**Handle 3DS not authenticated (Default branch):**

1. Add a **Notify** step and connect it to the **Default** branch.
2. Name it "Notify -- 3DS Not Authenticated".

**Add the Authorize step on the Authenticated path:**

1. Click **Add step** and select **Authorize** under Actions.
2. Draw a connection line from the **Authenticated** branch to the Authorize step.
3. Select your payment provider. Name the step "Authorize (Medium Risk + 3DS)".

Add Notify steps for all four Authorize outcomes:

| Outcome       | Notify Step Name                           |
| ------------- | ------------------------------------------ |
| **Completed** | Notify -- Med Risk Authorization Completed |
| **Paused**    | Notify -- Med Risk Authorization Paused    |
| **Requested** | Notify -- Med Risk Authorization Requested |
| **Updated**   | Notify -- Med Risk Authorization Updated   |

Connect each outcome to its Notify step.

### 10. Save and Activate

1. Click **Save**.
2. Review the entire canvas to confirm:
   - Every Fraud Check outcome has a connected step.
   - The risk-level Condition has all three branches (Low Risk, Medium Risk, Default) connected.
   - On the low-risk path, every Authorize outcome has a Notify step.
   - On the medium-risk path, every 3DS outcome has a connected step, the 3DS Condition has both branches connected, and every Authorize outcome has a Notify step.
   - On the high-risk path, the Default branch connects to a Notify step.
3. Activate the workflow when ready.

## Understanding the Outcome Paths

This workflow has **thirteen terminal paths**. Every path ends with a Notify step.

### Fraud Check outcomes (2 paths)

| Path                                 | What Happened                                                     |
| ------------------------------------ | ----------------------------------------------------------------- |
| Fraud Check --> Paused --> Notify    | The fraud provider needs additional information or manual review. |
| Fraud Check --> Requested --> Notify | The fraud check was submitted but the result is pending.          |

### High-risk path (1 path)

| Path                                                           | What Happened                                                                   |
| -------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Fraud Check --> Completed --> Condition --> Default --> Notify | The fraud score is too high. The transaction is rejected without authorization. |

### Low-risk path (4 paths)

| Path                                                    | What Happened                                          |
| ------------------------------------------------------- | ------------------------------------------------------ |
| ... --> Low Risk --> Authorize --> Completed --> Notify | Low-risk transaction authorized successfully.          |
| ... --> Low Risk --> Authorize --> Paused --> Notify    | Low-risk authorization waiting for an external action. |
| ... --> Low Risk --> Authorize --> Requested --> Notify | Low-risk authorization submitted for processing.       |
| ... --> Low Risk --> Authorize --> Updated --> Notify   | Low-risk authorization received a status update.       |

### Medium-risk path (6 paths)

| Path                                                                                                             | What Happened                                                          |
| ---------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| ... --> Medium Risk --> 3DS --> Paused --> Notify                                                                | The cardholder is completing the 3DS challenge.                        |
| ... --> Medium Risk --> 3DS --> Requested --> Notify                                                             | The 3DS challenge is being processed by the provider.                  |
| ... --> Medium Risk --> 3DS --> Completed --> Condition --> Default --> Notify                                   | 3DS finished but authentication failed. Transaction is not authorized. |
| ... --> Medium Risk --> 3DS --> Completed --> Condition --> Authenticated --> Authorize --> Completed --> Notify | Authenticated and authorized successfully.                             |
| ... --> Medium Risk --> 3DS --> Completed --> Condition --> Authenticated --> Authorize --> Paused --> Notify    | Authenticated but authorization waiting for an external action.        |
| ... --> Medium Risk --> 3DS --> Completed --> Condition --> Authenticated --> Authorize --> Requested --> Notify | Authenticated and authorization submitted for processing.              |
| ... --> Medium Risk --> 3DS --> Completed --> Condition --> Authenticated --> Authorize --> Updated --> Notify   | Authenticated and authorization received a status update.              |

## Why Risk-Based Routing Matters

Applying 3DS to every transaction would maximize security but hurt conversion rates. Rejecting anything above the lowest risk would decline legitimate transactions. Risk-based routing lets you apply the right level of security for each transaction:

| Risk Level | Action             | Customer Experience              | Security                                       |
| ---------- | ------------------ | -------------------------------- | ---------------------------------------------- |
| **Low**    | Authorize directly | Fastest checkout, no extra steps | Relies on fraud score and provider-side checks |
| **Medium** | 3DS then Authorize | One extra authentication step    | Cardholder verification plus liability shift   |
| **High**   | Reject immediately | Transaction blocked              | Highest protection against fraud               |

This approach concentrates friction where it matters most -- on the transactions that need additional verification -- while keeping the checkout smooth for your trusted customers.

## Customization Ideas

- **Adjust risk thresholds.** Fine-tune the Low Risk and Medium Risk condition rules to match your business's risk tolerance. Start with conservative thresholds and widen them as you gain confidence.
- **Add more risk tiers.** You can add additional condition branches for finer-grained routing -- for example, a "Very Low Risk" tier that skips certain provider checks.
- **Add a Fraud Update step.** After the Authorize Completed outcome on both the low-risk and medium-risk paths, add a Fraud Update step to feed the authorization result back to the fraud provider. This improves future fraud scoring accuracy.
- **Route to different providers by risk.** Use different provider selections on the low-risk and medium-risk Authorize steps to route transactions to providers best suited for each risk tier.

## Next Steps

- Review the [Simple Authorization](./simple-authorization) example if you want to start with a basic workflow and build up from there.
- See [Authorization with Fraud Screening](./authorization-with-fraud) or [Authorization with 3D Secure](./authorization-with-3ds) for simpler workflows that use just one of the two security steps.
