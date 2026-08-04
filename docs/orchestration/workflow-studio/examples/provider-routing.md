---
title: Provider Routing with Conditions
---

# Provider Routing with Conditions

This example walks you through building an authorization workflow that routes transactions to different payment providers based on attributes like card brand, country, currency, or amount. By using a Condition step before authorization, you can optimize processing costs, improve approval rates, or comply with regional requirements by sending each transaction to the most appropriate provider.

## When to use this workflow

Use this pattern when:

- You process payments through multiple payment providers and want to route transactions to the best provider based on transaction attributes.
- You want to optimize processing costs by sending certain card brands or currencies to providers with lower fees.
- You need to comply with regional requirements that mandate processing through a local provider for certain countries.
- You want to improve approval rates by routing transactions to providers that perform better for specific payment method types or regions.

## What you will build

A workflow that:

1. Receives an authorization request (Start Authorize trigger).
2. Evaluates the transaction attributes (Condition step with multiple branches).
3. Routes to the appropriate provider: Provider A, Provider B, or a default provider (separate Authorize steps, each configured with a different provider).
4. Sends a notification on every terminal path so your integration always receives a response.

## Workflow diagram

```
                         Start Authorize
                               |
                    Condition (Route Transaction)
                   /           |            \
             Visa/MC       AMEX           Default
                |             |              |
        Authorize (A)   Authorize (B)   Authorize (C)
        /  |  \   \     /  |  \   \     /  |  \   \
       C   P  Req  U   C   P  Req  U   C   P  Req  U
       |   |   |   |   |   |   |   |   |   |   |   |
       N   N   N   N   N   N   N   N   N   N   N   N
```

Where **C** = Completed, **P** = Paused, **Req** = Requested, **U** = Updated, and **N** = Notify.
**Authorize (A)** = Provider A, **Authorize (B)** = Provider B, **Authorize (C)** = Default provider.

## Step-by-step instructions

### Step 1: Create the workflow

1. Navigate to **Workflow Studio** in the admin portal sidebar.
2. Click **Create Workflow**.
3. Select **Start Authorize** from the trigger dropdown.
4. Name the workflow something descriptive, such as "Provider Routing by Card Brand".
5. Click **Create**.

The **Start Authorize** trigger step appears on the canvas. It fires whenever an authorization request is received through the API.

### Step 2: Add the Condition step

1. Click the **Add step** button on the canvas.
2. Under **Actions**, select **Condition**.
3. Draw a connection line from the **Start Authorize** trigger to the **Condition** step.
4. Click on the Condition step to open its settings panel.
5. Name the step "Route Transaction".

### Step 3: Configure the condition branches

In the Condition step settings panel, define branches for each routing rule. This example routes by card brand, but you can use any available field -- country, currency, amount, payment method, or a combination.

**Add the "Visa / Mastercard" branch:**

1. Click **Add Condition** in the settings panel.
2. Name the branch "Visa / Mastercard".
3. Define the rule:
   - Select the **card brand** field from the dropdown.
   - Choose the **is one of** operator.
   - Enter **Visa, Mastercard** as the values.

This branch matches transactions where the card brand is Visa or Mastercard.

**Add the "AMEX" branch:**

1. Click **Add Condition** again.
2. Name the branch "AMEX".
3. Define the rule:
   - Select the **card brand** field from the dropdown.
   - Choose the **equals** operator.
   - Set the value to **American Express**.

This branch matches American Express transactions.

**Default branch:**

The Default branch catches all transactions that do not match either custom branch -- for example, other card brands, alternative payment methods, or any unexpected values. This branch routes to your default provider.

> **Tip:** Place the most specific branches at the top. Branches are evaluated from top to bottom, and the first matching branch is followed. In this example, "AMEX" is more specific than "Visa / Mastercard", but since they do not overlap, the order does not matter. If your branches could overlap, put the more specific one first.

### Step 4: Add Authorize steps with different providers

Each branch needs its own Authorize step configured with the appropriate provider.

**Provider A (Visa / Mastercard):**

1. Click **Add step** and select **Authorize** from the actions menu.
2. Draw a connection line from the **Visa / Mastercard** branch to this Authorize step.
3. Click the Authorize step to open its settings panel.
4. Select **Provider A** from the provider dropdown -- this is the provider optimized for Visa and Mastercard transactions.
5. Configure any additional provider-specific settings as needed.

**Provider B (AMEX):**

1. Add another **Authorize** step to the canvas.
2. Draw a connection line from the **AMEX** branch to this Authorize step.
3. Open its settings panel and select **Provider B** from the provider dropdown -- this is the provider optimized for American Express transactions.
4. Configure any additional settings.

**Default Provider:**

1. Add a third **Authorize** step to the canvas.
2. Draw a connection line from the **Default** branch to this Authorize step.
3. Open its settings panel and select your **default provider** from the provider dropdown -- this is the fallback provider for all other transaction types.
4. Configure any additional settings.

### Step 5: Add Notify steps on all outcomes

Each Authorize step produces four outcomes: **Completed**, **Paused**, **Requested**, and **Updated**. Every outcome on every Authorize step needs a Notify step.

For each of the three Authorize steps, add four Notify steps:

1. **Completed** -- Configure the notification to confirm the authorization was successful. You might include which provider handled the transaction.
2. **Paused** -- Configure it to indicate the authorization is waiting for an external event (such as a shopper action).
3. **Requested** -- Configure it to indicate the authorization has been submitted and is pending.
4. **Updated** -- Configure it to indicate the authorization received a status update.

Repeat for all three Authorize steps (Provider A, Provider B, and the default provider). This gives you twelve Notify steps in total.

> **Tip:** You can reuse similar notification content across providers, but consider including the provider name in each notification so your integration knows which provider handled the transaction.

### Step 6: Review and save

Verify your workflow:

- The **Start Authorize** trigger is connected to the **Condition** step.
- The Condition step has a **Visa / Mastercard** branch, an **AMEX** branch, and a **Default** branch.
- Each branch leads to its own **Authorize** step with a different provider selected.
- All four outcomes (Completed, Paused, Requested, Updated) on each Authorize step connect to Notify steps.
- Every path through the workflow ends with a Notify step -- there are no dead ends.

Click **Save** to save the workflow. When you are ready, activate it to make it live.

## Understanding the outcome paths

| Path                 | What happens                                                           | Notification content                                   |
| -------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------ |
| Visa/MC -- Completed | Visa or Mastercard transaction authorized successfully via Provider A. | Confirm successful authorization via Provider A.       |
| Visa/MC -- Paused    | Authorization via Provider A is waiting for shopper action.            | Indicate authorization is paused.                      |
| Visa/MC -- Requested | Authorization submitted to Provider A, pending response.               | Indicate authorization is processing.                  |
| Visa/MC -- Updated   | Provider A sent a status update for the authorization.                 | Report the status update.                              |
| AMEX -- Completed    | American Express transaction authorized successfully via Provider B.   | Confirm successful authorization via Provider B.       |
| AMEX -- Paused       | Authorization via Provider B is waiting for shopper action.            | Indicate authorization is paused.                      |
| AMEX -- Requested    | Authorization submitted to Provider B, pending response.               | Indicate authorization is processing.                  |
| AMEX -- Updated      | Provider B sent a status update for the authorization.                 | Report the status update.                              |
| Default -- Completed | Other transaction authorized successfully via the default provider.    | Confirm successful authorization via default provider. |
| Default -- Paused    | Authorization via default provider is waiting for shopper action.      | Indicate authorization is paused.                      |
| Default -- Requested | Authorization submitted to default provider, pending response.         | Indicate authorization is processing.                  |
| Default -- Updated   | Default provider sent a status update for the authorization.           | Report the status update.                              |

## Routing by other attributes

The card brand example above is just one approach. You can route transactions using any field available in the Condition step rule builder. Common routing strategies include:

**By country:**
Route transactions to a local provider based on the shopper's country or the issuing country of the card. This can improve approval rates and comply with local processing requirements.

**By currency:**
Route EUR transactions to a European provider, USD transactions to a US provider, and so on. This can reduce currency conversion fees and improve settlement times.

**By amount:**
Route high-value transactions (amount `>=` a threshold) to a provider with better fraud protection, while sending lower-value transactions to a provider with lower fees.

**By combination:**
Chain multiple Condition steps for complex routing. For example, first check the country, then check the card brand within each country branch.

```
Start Authorize --> Condition (Country)
    |-- EU --> Condition (Card Brand)
    |              |-- Visa/MC --> Authorize (EU Provider A)
    |              \-- Default --> Authorize (EU Provider B)
    |-- US --> Authorize (US Provider)
    \-- Default --> Authorize (Global Provider)
```

## Best practices

- **Always handle the Default branch.** Unexpected card brands, new payment methods, or missing data can cause transactions to fall through to Default. Make sure the default provider can handle any transaction type.
- **Notify on every terminal path.** With three Authorize steps and four outcomes each, that is twelve Notify steps. It is tempting to skip some, but every path needs a Notify step so your integration always receives a response.
- **Name your steps clearly.** With multiple Authorize steps on the canvas, give each one a descriptive name (such as "Authorize via Provider A" or "Authorize -- AMEX") so the workflow is easy to read.
- **Test each branch.** After activating the workflow, send test transactions that match each condition branch to verify the routing works as expected.
- **Monitor provider performance.** Use the execution monitoring tools to track which providers are handling which transactions and how they perform. Adjust your routing rules over time based on approval rates and processing costs.
- **Keep conditions maintainable.** If you have many providers and complex routing rules, consider splitting the logic across multiple chained Condition steps rather than putting everything in one step with many branches.

## Next steps

- Learn more about the [Condition](/docs/orchestration/workflow-studio/conditions/index) step and how to build rules.
- Learn more about the [Authorize](/docs/orchestration/workflow-studio/actions#authorize) action, provider selection, and outcomes.
- See the [Conditional Refund](/docs/orchestration/workflow-studio/overview) example for a workflow that branches based on transaction attributes.
- See the [Event-Driven Auto-Capture](/docs/orchestration/workflow-studio/overview) example for a workflow that reacts to asynchronous payment events.
