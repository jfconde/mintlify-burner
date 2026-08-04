---
title: Event-Driven Auto-Capture
---
# Event-Driven Auto-Capture

This example walks you through building a workflow that automatically captures a payment when an asynchronous authorization succeeds. Instead of waiting for your integration to send a separate capture request, this workflow listens for payment status updates and triggers the capture automatically.

## When to use this workflow

Use this pattern when:

* You work with payment methods that authorize asynchronously (such as bank transfers, redirect-based methods, or installment payments) where the authorization result arrives later via a provider callback.
* You want to capture payments immediately upon successful authorization without requiring your back-end integration to send a separate capture request.
* You want to reduce the delay between authorization and capture for asynchronous payment flows.

## What you will build

A workflow that:

1. Listens for payment status update events (Handle Payment Status Update trigger).
2. Checks whether the status update indicates a successful authorization (Condition step).
3. If the authorization succeeded, captures the payment automatically (Capture step).
4. If the status is anything else, sends a notification without capturing (Notify step on the Default branch).
5. Sends a notification on every terminal path.

## Workflow diagram

```
         Handle Payment Status Update
                     |
            Condition (Status Check)
             /                  \
   authorizeSuccessful        Default
            |                    |
         Capture               Notify
       /  |   \   \
      C   P   Req  U
      |   |    |   |
      N   N    N   N
```

Where **C** = Completed, **P** = Paused, **Req** = Requested, **U** = Updated, and **N** = Notify.

## Step-by-step instructions

### Step 1: Create the workflow

1. Navigate to **Workflow Studio** in the admin portal sidebar.
2. Click **Create Workflow**.
3. Select **Handle Payment Status Update** from the trigger dropdown.
4. Name the workflow something descriptive, such as "Auto-Capture on Authorization Success".
5. Click **Create**.

The **Handle Payment Status Update** trigger step appears on the canvas. This trigger fires automatically whenever a payment status update event is received from a provider -- no API call from your integration is required.

### Step 2: Add the Condition step

You need to check whether the status update indicates a successful authorization before capturing.

1. Click the **Add step** button on the canvas.
2. Under **Actions**, select **Condition**.
3. Draw a connection line from the **Handle Payment Status Update** trigger to the **Condition** step.
4. Click on the Condition step to open its settings panel.
5. Name the step "Check Authorization Status".

### Step 3: Configure the condition branches

In the Condition step settings panel, define a branch that matches successful authorizations.

**Add the "Authorization Successful" branch:**

1. Click **Add Condition** in the settings panel.
2. Name the branch "Authorization Successful".
3. Define the rule:
   * Select the **payment status** field from the dropdown.
   * Choose the **equals** operator.
   * Set the value to **authorizeSuccessful**.

This branch matches when the status update indicates the authorization has been completed successfully.

**Default branch:**

The Default branch catches all other status updates -- declined authorizations, pending statuses, capture updates, refund updates, and any other status that is not a successful authorization. For this workflow, the Default branch simply sends a notification without taking further action.

### Step 4: Add the Capture step on the Authorization Successful branch

1. Click **Add step** and select **Capture** from the actions menu.
2. Draw a connection line from the **Authorization Successful** branch on the Condition step to the Capture step.

The Capture step does not require provider selection -- it automatically captures with the same provider that handled the original authorization.

### Step 5: Add Notify on the Default branch

1. Add a **Notify** step to the canvas.
2. Draw a connection line from the **Default** branch on the Condition step to this Notify step.
3. Configure the notification to inform your integration that a payment status update was received but no auto-capture was triggered. You might include the status value so your integration can decide whether further action is needed.

### Step 6: Add Notify steps on all Capture outcomes

The Capture step produces four outcomes: **Completed**, **Paused**, **Requested**, and **Updated**. Each one needs a Notify step.

1. Add a **Notify** step and connect it to the **Completed** outcome. Configure it to confirm the auto-capture was successful.
2. Add a **Notify** step and connect it to the **Paused** outcome. Configure it to indicate the capture is paused and awaiting further action.
3. Add a **Notify** step and connect it to the **Requested** outcome. Configure it to indicate the capture has been submitted and is pending.
4. Add a **Notify** step and connect it to the **Updated** outcome. Configure it to indicate the capture received a status update.

### Step 7: Review and save

Verify your workflow:

* The **Handle Payment Status Update** trigger is connected to the **Condition** step.
* The Condition step has an **Authorization Successful** branch and a **Default** branch.
* The Authorization Successful branch leads to a **Capture** step.
* All four Capture outcomes (Completed, Paused, Requested, Updated) connect to Notify steps.
* The Default branch connects directly to a Notify step.
* Every path through the workflow ends with a Notify step.

Click **Save** to save the workflow. When you are ready, activate it to make it live.

## Understanding the outcome paths

| Path                                          | What happens                                                                                                   | Notification content                                       |
| --------------------------------------------- | -------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| Authorization Successful -- Capture Completed | The authorization succeeded asynchronously and the auto-capture completed. Funds have been transferred.        | Confirm successful auto-capture.                           |
| Authorization Successful -- Capture Paused    | The authorization succeeded but the capture is waiting for an external event.                                  | Indicate the auto-capture is paused.                       |
| Authorization Successful -- Capture Requested | The authorization succeeded and the capture has been submitted to the provider.                                | Indicate the auto-capture is processing.                   |
| Authorization Successful -- Capture Updated   | The authorization succeeded and the capture received a status update.                                          | Report the capture status update.                          |
| Default                                       | The status update was not a successful authorization (e.g., declined, pending, or a non-authorization status). | Inform integration of the status update without capturing. |

## Important considerations

* **This workflow only runs on status update events.** It does not replace your standard capture workflow. If you also process synchronous authorizations that do not generate a status update event, you still need a separate capture flow for those.
* **Filter carefully.** The Condition step ensures you only auto-capture on successful authorizations. Without this filter, you might attempt to capture payments that were declined or are still pending.
* **Duplicate events.** Payment status update events can sometimes arrive more than once. The platform handles deduplication, but be aware that your notification content should be idempotent -- meaning it produces the correct result even if received twice.
* **Event timing.** Asynchronous status updates can arrive minutes, hours, or even days after the original authorization request, depending on the payment method and provider. This workflow handles them whenever they arrive.

## Best practices

* **Always include the Default branch.** The Handle Payment Status Update trigger fires for all payment status changes, not just authorizations. The Default branch ensures your workflow handles non-authorization updates gracefully.
* **Notify on every terminal path.** Whether the auto-capture succeeds, fails, or is not triggered at all, your integration needs to know the result.
* **Name your condition branches clearly.** "Authorization Successful" is much more readable than a generic branch name, especially when reviewing the workflow later.
* **Test with asynchronous payment methods.** This workflow is specifically designed for payment methods where the authorization result arrives asynchronously. Test with the relevant payment methods to confirm the status values match your condition rules.

## Next steps

* Learn more about the [Handle Payment Status Update](/docs/orchestration/workflow-studio/triggers#handle-payment-status-update) trigger and when it fires.
* Learn more about the [Capture](/docs/orchestration/workflow-studio/actions#capture) action and its outcomes.
* Learn more about the [Condition](/docs/orchestration/workflow-studio/conditions/index) step and how to build rules.
* See the [Capture with Fraud Update](/docs/orchestration/workflow-studio/overview) example to add fraud reporting to your capture flows.
