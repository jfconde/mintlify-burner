---
title: Capture with Fraud Update
---
# Capture with Fraud Update

This example walks you through building a capture workflow that sends the transaction outcome back to your fraud provider after a successful capture. By reporting capture results to the fraud provider, you help it refine its risk models and produce more accurate fraud assessments for future transactions.

## When to use this workflow

Use this pattern when:

* You process payments through a fraud provider that supports outcome reporting (such as post-authorization or post-capture feedback).
* You want to improve the accuracy of future fraud scores by feeding real transaction results back to the provider.
* You already have a Fraud Check step in your authorization workflow and want to close the feedback loop at capture time.

## What you will build

A workflow that:

1. Receives a capture request (Start Capture trigger).
2. Captures the previously authorized payment (Capture step).
3. On the Completed outcome, sends the result to the fraud provider (Fraud Update step).
4. Sends a notification on every terminal path so your integration always receives a response.

## Workflow diagram

```
                  Start Capture
                       |
                    Capture
                  /   |    \       \
          Completed  Paused  Requested  Updated
              |        |        |          |
        Fraud Update  Notify   Notify    Notify
        /    |    \
  Completed Paused Requested
       |      |       |
     Notify  Notify  Notify
```

## Step-by-step instructions

### Step 1: Create the workflow

1. Navigate to **Workflow Studio** in the admin portal sidebar.
2. Click **Create Workflow**.
3. Select **Start Capture** from the trigger dropdown.
4. Name the workflow something descriptive, such as "Capture with Fraud Update".
5. Click **Create**.

The **Start Capture** trigger step appears on the canvas. This is the entry point -- it fires whenever a capture request is received through the API.

### Step 2: Add the Capture step

1. Click the **Add step** button on the canvas.
2. Under **Actions**, select **Capture**.
3. Draw a connection line from the **Start Capture** trigger to the **Capture** step.

The Capture step does not require provider selection -- it automatically captures with the same provider that handled the original authorization.

### Step 3: Add Fraud Update on the Completed outcome

The Completed outcome means the capture finished successfully and funds have been transferred. This is the point where you want to inform the fraud provider of the outcome.

1. Click **Add step** and select **Fraud Update** from the actions menu.
2. Draw a connection line from the **Completed** outcome on the Capture step to the **Fraud Update** step.
3. Click on the Fraud Update step to open its settings panel.
4. In the settings panel, select the **fraud provider configuration** from the provider dropdown. Choose the same provider that performed the original Fraud Check during authorization.
5. Close the settings panel.

### Step 4: Handle the Fraud Update outcomes

The Fraud Update step produces three outcomes: **Completed**, **Paused**, and **Requested**. Each needs a Notify step.

1. Add a **Notify** step and connect it to the **Completed** outcome of the Fraud Update. Configure it to confirm that both the capture and fraud update succeeded.
2. Add a **Notify** step and connect it to the **Paused** outcome. Configure it to indicate the fraud update is waiting for an external response.
3. Add a **Notify** step and connect it to the **Requested** outcome. Configure it to indicate the fraud update has been submitted and is pending.

### Step 5: Handle the remaining Capture outcomes

The Capture step has three additional outcomes beyond Completed that need their own Notify steps.

**Paused outcome:**
The capture is waiting for an external event before it can complete. Add a **Notify** step and connect it to the **Paused** outcome. Configure it to indicate the capture is paused and awaiting further action.

**Requested outcome:**
The capture has been submitted to the provider but a final result is not yet available. Add a **Notify** step and connect it to the **Requested** outcome. Configure it to indicate the capture is pending.

**Updated outcome:**
The capture received a status update from the provider. Add a **Notify** step and connect it to the **Updated** outcome. Configure it to indicate the capture status has been updated.

### Step 6: Review and save

Verify your workflow:

* The **Start Capture** trigger is connected to the **Capture** step.
* The **Completed** outcome of Capture connects to the **Fraud Update** step.
* All three outcomes of Fraud Update (Completed, Paused, Requested) connect to Notify steps.
* The Paused, Requested, and Updated outcomes of Capture each connect directly to their own Notify steps.
* Every path through the workflow ends with a Notify step -- there are no dead ends.

Click **Save** to save the workflow. When you are ready, activate it to make it live.

## Understanding the outcome paths

| Path                                           | What happens                                                                                    | Notification content                                               |
| ---------------------------------------------- | ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| Capture Completed, then Fraud Update Completed | The capture succeeded and the fraud provider acknowledged the result. This is the ideal path.   | Confirm successful capture and fraud update.                       |
| Capture Completed, then Fraud Update Paused    | The capture succeeded but the fraud update is waiting for an external response.                 | Confirm capture success; note that the fraud update is pending.    |
| Capture Completed, then Fraud Update Requested | The capture succeeded but the fraud update has been submitted and is not yet acknowledged.      | Confirm capture success; note that the fraud update is processing. |
| Capture Paused                                 | The capture itself is waiting for an external event. The fraud update has not been reached yet. | Inform integration that the capture is paused.                     |
| Capture Requested                              | The capture has been submitted but is pending. The fraud update has not been reached yet.       | Inform integration that the capture is processing.                 |
| Capture Updated                                | The capture received a status update from the provider.                                         | Inform integration of the capture status update.                   |

## Best practices

* **Select the correct fraud provider.** In the Fraud Update settings panel, choose the same fraud provider configuration that performed the original Fraud Check during authorization. This ensures the feedback reaches the correct risk model.
* **Always notify on every terminal path.** Even if the fraud update fails or is pending, your integration needs to know the capture result. Every branch should end with a Notify step.
* **Fraud Update only on Completed.** Place the Fraud Update step only on the Completed outcome of Capture. The other outcomes (Paused, Requested, Updated) do not yet have a final capture result, so there is nothing meaningful to report to the fraud provider.
* **Keep the workflow focused.** This workflow handles capture and fraud feedback. If you need additional logic (such as conditional routing), consider adding a Condition step before the Capture action.

## Next steps

* Learn more about the [Capture](/docs/orchestration/workflow-studio/actions#capture) action and its outcomes.
* Learn more about the [Fraud Update](/docs/orchestration/workflow-studio/actions#fraud-check) action and provider selection.
* See the [Conditional Refund](/docs/orchestration/workflow-studio/overview) example for a workflow that uses conditions to branch based on refund type.
