---
title: Conditional Refund
---

# Conditional Refund

This example walks you through building a refund workflow that uses a Condition step to handle full refunds and partial refunds differently. By branching early in the workflow, you can apply separate logic, routing, or notification content depending on the refund type.

## When to use this workflow

Use this pattern when:

- You need to distinguish between full and partial refunds and handle them with different logic.
- You want to send different notification content depending on the refund type (for example, a confirmation message for full refunds and a remaining-balance message for partial refunds).
- You want to apply different validation or routing rules based on the refund amount.

## What you will build

A workflow that:

1. Receives a refund request (Start Refund trigger).
2. Evaluates whether the refund is a full refund or a partial refund (Condition step).
3. Processes the refund on each branch (Refund step).
4. Sends a notification on every terminal path so your integration always receives a response.

## Workflow diagram

```
                     Start Refund
                          |
                   Condition (Refund Type)
                    /                \
             Full Refund           Default
            (amount = full)     (partial refund)
                 |                    |
              Refund              Refund
           /  |   \   \        /  |   \   \
          C   P   Req  U      C   P   Req  U
          |   |    |   |      |   |    |   |
          N   N    N   N      N   N    N   N
```

Where **C** = Completed, **P** = Paused, **Req** = Requested, **U** = Updated, and **N** = Notify.

## Step-by-step instructions

### Step 1: Create the workflow

1. Navigate to **Workflow Studio** in the admin portal sidebar.
2. Click **Create Workflow**.
3. Select **Start Refund** from the trigger dropdown.
4. Name the workflow something descriptive, such as "Conditional Refund".
5. Click **Create**.

The **Start Refund** trigger step appears on the canvas.

### Step 2: Add the Condition step

1. Click the **Add step** button on the canvas.
2. Under **Actions**, select **Condition**.
3. Draw a connection line from the **Start Refund** trigger to the **Condition** step.
4. Click on the Condition step to open its settings panel.
5. Give the step a descriptive name, such as "Check Refund Type".

### Step 3: Configure the condition branches

In the Condition step settings panel, define the branches that distinguish full refunds from partial refunds.

**Add the "Full Refund" branch:**

1. Click **Add Condition** in the settings panel.
2. Name the branch "Full Refund".
3. Define the rule:
   - Select the **refund amount** field from the dropdown.
   - Choose the **equals** operator.
   - Set the value to the **original captured amount** field.

This branch matches when the refund amount equals the full captured amount.

**Default branch:**

The Default branch automatically catches any refund that does not match the "Full Refund" condition -- in this case, partial refunds. You do not need to configure any rules for it; it is always present.

> **Tip:** Give your Default branch a clear purpose. In this workflow, the Default branch represents partial refunds. You can add a note or label to make this intent clear on the canvas.

### Step 4: Add the Refund step on the Full Refund branch

1. Click **Add step** and select **Refund** from the actions menu.
2. Draw a connection line from the **Full Refund** branch on the Condition step to this Refund step.
3. Click on the Refund step to open its settings panel and review any available settings.

### Step 5: Add the Refund step on the Default branch

1. Add another **Refund** step to the canvas.
2. Draw a connection line from the **Default** branch on the Condition step to this second Refund step.
3. Open its settings panel and review the settings. You can configure different settings for partial refunds if needed.

### Step 6: Add Notify steps on all outcomes

Each Refund step produces four outcomes: **Completed**, **Paused**, **Requested**, and **Updated**. Every outcome on both Refund steps needs a Notify step.

**For the Full Refund path:**

1. Add a **Notify** step and connect it to the **Completed** outcome. Configure the notification to confirm the full refund was processed.
2. Add a **Notify** step for the **Paused** outcome. Configure it to indicate the refund is paused.
3. Add a **Notify** step for the **Requested** outcome. Configure it to indicate the refund is pending.
4. Add a **Notify** step for the **Updated** outcome. Configure it to indicate a status update was received.

**For the Default (partial refund) path:**

1. Add a **Notify** step and connect it to the **Completed** outcome. Configure the notification to confirm the partial refund was processed and include the remaining balance if relevant.
2. Add a **Notify** step for the **Paused** outcome.
3. Add a **Notify** step for the **Requested** outcome.
4. Add a **Notify** step for the **Updated** outcome.

> **Tip:** You can tailor the notification content on each branch. For example, the Full Refund Completed notification might say "Full refund processed", while the partial refund Completed notification might say "Partial refund processed -- remaining balance available for further refunds".

### Step 7: Review and save

Verify your workflow:

- The **Start Refund** trigger is connected to the **Condition** step.
- The Condition step has a **Full Refund** branch and a **Default** branch.
- Each branch leads to its own **Refund** step.
- All four outcomes (Completed, Paused, Requested, Updated) on each Refund step connect to a Notify step.
- Every path through the workflow ends with a Notify step.

Click **Save** to save the workflow. When you are ready, activate it to make it live.

## Understanding the outcome paths

| Path                           | What happens                                                                              | Notification content                                                     |
| ------------------------------ | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| Full Refund -- Completed       | The full refund was processed successfully. The entire captured amount has been returned. | Confirm full refund success.                                             |
| Full Refund -- Paused          | The full refund is waiting for an external event.                                         | Indicate the full refund is paused.                                      |
| Full Refund -- Requested       | The full refund has been submitted and is pending.                                        | Indicate the full refund is processing.                                  |
| Full Refund -- Updated         | The full refund received a status update from the provider.                               | Report the status update.                                                |
| Default (Partial) -- Completed | The partial refund was processed successfully.                                            | Confirm partial refund success; include remaining balance if applicable. |
| Default (Partial) -- Paused    | The partial refund is waiting for an external event.                                      | Indicate the partial refund is paused.                                   |
| Default (Partial) -- Requested | The partial refund has been submitted and is pending.                                     | Indicate the partial refund is processing.                               |
| Default (Partial) -- Updated   | The partial refund received a status update.                                              | Report the status update.                                                |

## Best practices

- **Use descriptive branch names.** Naming your condition branch "Full Refund" (rather than something generic like "Branch 1") makes the workflow immediately readable for anyone reviewing it.
- **Always handle the Default branch.** Even if you expect most refunds to match your custom condition, the Default branch catches everything else. Never leave it disconnected.
- **Customize notification content per branch.** The main benefit of conditional branching is that you can send different, more specific information to your integration based on the refund type.
- **Keep conditions simple.** A single condition comparing the refund amount to the captured amount is sufficient for this use case. If you need more complex logic (such as routing by currency or payment method), consider adding additional condition branches or chaining multiple Condition steps.

## Next steps

- Learn more about the [Condition](/docs/orchestration/workflow-studio/conditions/index) step and how to build rules.
- Learn more about the [Refund](/docs/orchestration/workflow-studio/actions#refund) action and its outcomes.
- See the [Capture with Fraud Update](/docs/orchestration/workflow-studio/overview) example for a workflow that adds fraud reporting after capture.
- See the [Provider Routing with Conditions](/docs/orchestration/workflow-studio/overview) example for a workflow that routes to different providers based on transaction attributes.
