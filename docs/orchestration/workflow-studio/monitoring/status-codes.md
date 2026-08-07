---
title: Status Codes Reference
---

# Status Codes Reference

Every workflow execution is assigned a status code that reflects its current state. Status codes are color-coded by severity so you can quickly identify whether an execution succeeded, failed, or is still in progress.

This page provides a complete reference of all 29 status codes.

## Green -- Success

Green status codes indicate that the operation completed successfully. These are terminal states for the corresponding action.

| Label | Code | Description |
|-------|------|-------------|
| **Created** | `created` | The payment record has been created successfully. |
| **Authorize Successful** | `authorizeSuccessful` | The authorization was approved by the provider. Funds are reserved on the customer's payment method. |
| **Confirm Successful** | `confirmSuccessful` | The payment confirmation completed successfully. |
| **Cancel Successful** | `cancelSuccessful` | The payment was successfully cancelled. The reserved funds have been released. |
| **Refund Successful** | `refundSuccessful` | The refund was processed successfully. Funds have been returned to the customer. |
| **Capture Successful** | `captureSuccessful` | The capture completed successfully. Reserved funds have been settled to the merchant. |
| **Fraud Score Successful** | `fraudScoreSuccessful` | The fraud check completed successfully and returned a risk assessment. |
| **Payout Successful** | `payoutSuccessful` | The payout was processed successfully. Funds have been transferred to the recipient. |

## Red -- Failure

Red status codes indicate that the operation failed. Review the execution details to understand the cause of the failure.

| Label | Code | Description |
|-------|------|-------------|
| **Authorize Failed** | `authorizeFailed` | The authorization was declined by the provider. No funds were reserved. |
| **Confirm Failed** | `confirmFailed` | The payment confirmation failed. |
| **Capture Failed** | `captureFailed` | The capture could not be completed. This may occur if the authorization has expired or was already cancelled. |
| **Cancel Failed** | `cancelFailed` | The cancellation could not be processed. This may occur if the payment has already been captured. |
| **Refund Failed** | `refundFailed` | The refund could not be processed. This may occur if the payment has not been captured or the refund amount exceeds the captured amount. |
| **Fraud Score Failed** | `fraudScoreFailed` | The fraud check could not be completed. The fraud scoring provider returned an error or was unreachable. |
| **Payout Failed** | `payoutFailed` | The payout could not be processed. This may be due to invalid recipient details or insufficient funds. |

## Yellow -- In Progress

Yellow status codes indicate that the operation has been initiated but is not yet complete. These are transitional states -- the execution is waiting for processing or an external response.

| Label | Code | Description |
|-------|------|-------------|
| **Authorize Requested** | `authorizeRequested` | The authorization request has been sent to the provider and is awaiting a response. |
| **Authorize Pending** | `authorizePending` | The authorization is being processed by the provider. This typically occurs when the provider needs additional time or is waiting for customer action (for example, 3DS verification). |
| **Confirm Requested** | `confirmRequested` | The confirmation request has been sent and is awaiting processing. |
| **Capture Requested** | `captureRequested` | The capture request has been sent to the provider and is awaiting processing. |
| **Capture Scheduled** | `captureScheduled` | The capture has been scheduled to execute at a later time. |
| **Capture Unscheduled** | `captureUnscheduled` | A previously scheduled capture has been unscheduled and will not execute automatically. |
| **Cancel Scheduled** | `cancelScheduled` | The cancellation has been scheduled to execute at a later time. |
| **Cancel Unscheduled** | `cancelUnscheduled` | A previously scheduled cancellation has been unscheduled and will not execute automatically. |
| **Cancel Requested** | `cancelRequested` | The cancellation request has been sent to the provider and is awaiting processing. |
| **Refund Requested** | `refundRequested` | The refund request has been sent to the provider and is awaiting processing. |
| **Fraud Score Requested** | `fraudScoreRequested` | The fraud check request has been sent to the fraud scoring provider and is awaiting a response. |
| **Fraud Score Pending** | `fraudScorePending` | The fraud check is being processed by the provider. This typically occurs when the provider needs additional time to calculate the risk score. |
| **Payout Requested** | `payoutRequested` | The payout request has been sent and is awaiting processing. |
| **Payout Pending** | `payoutPending` | The payout is being processed. This typically occurs when the payment provider needs additional time to complete the transfer. |

## Status code lifecycle

A typical payment operation moves through status codes in a predictable sequence:

1. **Requested** -- The operation has been submitted (for example, `authorizeRequested`).
2. **Pending** (if applicable) -- The provider is processing the request (for example, `authorizePending`).
3. **Successful** or **Failed** -- The operation reaches a terminal state (for example, `authorizeSuccessful` or `authorizeFailed`).

Some operations also support **Scheduled** and **Unscheduled** states, which indicate that the operation is queued for future execution or has been removed from the queue.


## Using status codes for monitoring

Status codes appear throughout the Workflow Studio monitoring interface:

- **Execution list** -- Each execution displays its current status code as a color-coded badge.
- **Execution detail** -- The step-by-step execution path shows the status code at each step.
- **Filtering** -- You can filter the execution list by status severity (green, yellow, or red) to focus on the executions that need attention.

For guidance on monitoring workflows and debugging failures, see [Monitoring Workflow Executions](./index).