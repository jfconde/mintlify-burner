---
title: Monitoring Workflow Executions
---

# Monitoring Workflow Executions

Workflow Studio provides a dedicated execution monitoring view where you can track the progress of your workflows, inspect individual executions, and debug failures. This page explains how to navigate the monitoring interface and make sense of execution data.


## Accessing the execution monitor

To view workflow executions:

1. Open **Workflow Studio** from the main navigation.
2. Navigate to the **Executions** section.
3. The execution list displays recent workflow runs, ordered by most recent first.

Each row in the list shows key information at a glance:

- **Execution ID** -- A unique identifier for the execution.
- **Workflow name** -- Which workflow was triggered.
- **Trigger** -- The trigger that started the execution.
- **Status** -- The current status of the execution (see [Understanding execution status](#understanding-execution-status) below).
- **Timestamp** -- When the execution started.


## Understanding execution status

Every execution has a status that indicates its current state. Statuses are color-coded for quick identification:

| Color | Meaning | Examples |
|-------|---------|----------|
| **Green** | The operation completed successfully. | Authorize Successful, Capture Successful, Refund Successful |
| **Yellow** | The operation is in progress or waiting for an external response. | Authorize Requested, Capture Scheduled, Fraud Score Pending |
| **Red** | The operation failed. | Authorize Failed, Capture Failed, Refund Failed |

A workflow execution may pass through multiple statuses as it progresses. For example, an authorization flow might move from **Authorize Requested** (yellow) to **Authorize Successful** (green) as the provider processes the payment.

For a complete list of all status codes and their meanings, see the [Status Codes Reference](./status-codes).

## Viewing execution details

Click on any execution in the list to open the execution detail view. This view shows:

### Execution path

A visual representation of which steps were executed and in what order. Each step in the path shows:

- The step type (trigger, action, or condition).
- The outcome that determined the next step.
- Whether the step succeeded or failed.


### Step details

Click on any step in the execution path to see its details:

- **Status** -- The result of that step.
- **Timing** -- When the step started and how long it took.
- **Input data** -- The data that was passed into the step.
- **Output data** -- The data that the step produced.


## Filtering and searching executions

The execution list supports several filtering options to help you find specific runs:

### Filter by status

Use the status filter to show only executions with a specific status. This is useful when you want to focus on failures or in-progress executions.


### Filter by workflow

Select a specific workflow to see only its executions. This helps when you manage multiple workflows and need to investigate a particular one.

### Filter by date range

Narrow the list to executions within a specific time period. Use the date range picker to set a start and end date.

### Search by execution ID

If you have a specific execution ID (for example, from an error log or support ticket), enter it in the search field to jump directly to that execution.


## Debugging failed workflows

When an execution fails, follow these steps to identify and resolve the issue:

### 1. Find the failed execution

Filter the execution list by **red** (failed) status to see all failed runs. Click on the execution you want to investigate.

### 2. Locate the failing step

In the execution path view, look for the step marked with a failure indicator. This is where the workflow encountered an error.

### 3. Inspect the step details

Open the failing step's details to review:

- **The status code** -- Tells you what type of failure occurred (for example, Authorize Failed vs. Capture Failed).
- **Input data** -- Verify that the data passed to the step was correct and complete.
- **Output data** -- Check for error messages or response codes from the provider.
- **Timing** -- Look for unusually long durations that might indicate timeouts.

### 4. Check preceding steps

Sometimes the root cause is not in the failing step itself. Review the steps that executed before the failure to make sure they produced the expected output.

### 5. Review your workflow design

If the failure is consistent, review the workflow in Workflow Studio:

- Confirm that all connection lines are correctly connected.
- Verify that condition rules are routing to the intended branches.
- Make sure the failing step's settings panel has the correct configuration.
- Check that every terminal path ends with a Notify step, so your integration always receives a response even on failure.


## Best practices for monitoring

- **Check executions regularly.** Review the execution list periodically to catch failures early, especially after deploying a new or updated workflow.
- **Set up Notify steps for all outcomes.** When every terminal path includes a Notify step, your integration is always informed of results. This makes it easier to correlate execution data with your own logs.
- **Use filters to focus.** When investigating an issue, filter by the specific workflow and time range to reduce noise.
- **Compare successful and failed runs.** If a workflow fails intermittently, compare the input data of successful and failed executions to identify what differs.
- **Review after workflow changes.** After editing a workflow, monitor its next few executions closely to confirm the changes work as expected.