---
title: Quick Start Guide
---
# Quick Start Guide

This guide walks you through building a simple authorization workflow from scratch. By the end, you will have a working workflow that authorizes a payment and sends notifications on every possible outcome.

## Prerequisites

Before you begin, make sure you have:

* Access to the admin portal with permissions to create workflows.
* At least one payment provider configured for authorization.

## What You Will Build

A workflow that:

1. Receives an authorization request.
2. Sends the authorization to a payment provider.
3. Handles every possible outcome (Completed, Paused, Requested, and Updated).
4. Sends a notification on each terminal path.

## Step 1: Create a New Workflow

1. Navigate to **Workflow Studio** in the admin portal sidebar.
2. Click **Create Workflow** to open a new, empty canvas.
3. Give your workflow a description to describe its purpose, such as "Basic Authorization".

## Step 2: Select a Payment Method

1. Navigate to **Workflows settings** in the top left of the canvas and select **Payment Options**.
2. Click **Select Payment Options** and select **Card**.
3. Click **Done** and **Continue Editing** to go back to the canvas with your selection.

## Step 3: Add the Start Authorize Trigger

Every workflow begins with a trigger. For an authorization workflow, you need the **Start Authorize** trigger.

1. Open the step menu by clicking the **Add step** button on the canvas.
2. Under **Triggers**, select **Start Authorize**.
3. The trigger step appears on the canvas. This is the entry point of your workflow -- it fires whenever an authorization request is received.

## Step 4: Add the Authorize Action

Next, add the action that performs the actual authorization.

1. Click the **Add step** button again.
2. Under **Actions**, select **Authorize**.
3. The Authorize step appears on the canvas.
4. Draw a connection line from the **Start Authorize** trigger to the **Authorize** action by dragging from the trigger's output to the action's input.

## Step 5: Configure the Authorize Step

The Authorize action supports provider selection, so you need to specify which payment provider should handle the authorization.

1. Click on the **Authorize** step to open its settings panel.
2. In the settings panel, select your payment provider from the provider dropdown.
3. Close the settings panel.

## Step 6: Handle the Outcomes

The Authorize action has four possible outcomes: **Completed**, **Paused**, **Requested**, and **Updated**. Each outcome represents a different result from the payment provider, and you need to handle every one of them.

You can see the four outcome connection points on the Authorize step. Each one needs a path leading to a next step.

### Handle the Completed Outcome

The **Completed** outcome means the authorization finished successfully.

1. Click **Add step** and select **Notify** from the Actions section.
2. Draw a connection line from the **Completed** outcome on the Authorize step to the new Notify step.
3. Click on the Notify step to open its settings panel and configure the notification for a successful authorization.

### Handle the Paused Outcome

The **Paused** outcome means the authorization is waiting for an external event (such as a shopper action) before it can continue.

1. Add another **Notify** step to the canvas.
2. Draw a connection line from the **Paused** outcome to this Notify step.
3. Configure the notification to indicate the authorization is paused and awaiting further action.

### Handle the Requested Outcome

The **Requested** outcome means the authorization has been submitted to the provider and is pending processing.

1. Add another **Notify** step to the canvas.
2. Draw a connection line from the **Requested** outcome to this Notify step.
3. Configure the notification to indicate the authorization has been requested.

### Handle the Updated Outcome

The **Updated** outcome means the authorization received a status update from the provider.

1. Add another **Notify** step to the canvas.
2. Draw a connection line from the **Updated** outcome to this Notify step.
3. Configure the notification to indicate the authorization has been updated.

## Step 7: Review Your Workflow

Your completed workflow should look like this:

```
Start Authorize
      |
  Authorize
   /  |  \   \
  C   P   R    U
  |   |   |    |
 N1  N2  N3   N4
```

Where **C** = Completed, **P** = Paused, **R** = Requested, **U** = Updated, and **N1-N4** are Notify steps.

Verify the following:

* The **Start Authorize** trigger is connected to the **Authorize** action.
* Every outcome on the Authorize step has a connection line leading to a Notify step.
* There are no unconnected outcomes or orphaned steps on the canvas.

## Step 8: Save and Activate

1. Click **Save** to save your workflow.
2. Review the workflow one more time to confirm all connection lines are correct.
3. When you are ready, activate the workflow to make it live.

> **Tip:** Always save your workflow before activating it. You can deactivate a workflow at any time if you need to make changes.

## Summary

You have built a complete authorization workflow that:

* Handles cards as a payment method.
* Starts when an authorization request is received (Start Authorize trigger).
* Sends the authorization to a payment provider (Authorize action with provider selection).
* Handles all four possible outcomes (Completed, Paused, Requested, Updated).
* Sends a notification on every terminal path (four Notify steps).

This pattern of handling every outcome with a Notify step is a best practice you should follow in all your workflows. It ensures your systems are always informed of the final result, regardless of which path the workflow takes.

## Next Steps

Now that you have built your first workflow, you can:

* Add a **Condition** step before the Authorize action to route requests to different providers based on rules.
* Insert a **Fraud Check** step before authorization to screen transactions.
* Add a **3DS** step to perform 3D Secure verification before authorizing.
* Explore other triggers like **Start Capture** and **Start Refund** to build workflows for the full payment lifecycle.