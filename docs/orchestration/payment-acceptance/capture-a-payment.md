# Introduction

By default, payments are automatically captured after authorization, without any delay. Alternatively, you can set a capture delay or manually capture a payment via the API or the Payrails Portal.

# Requirements

- The payment you want to capture must be authorized
- Have [notifications](/docs/orchestration/payment-acceptance/receive-notifications) set up if you wish to receive them for capture results

# Capture a payment via API

## 1. Get the execution ID

To capture a payment, you’ll need the ID of the execution. You can find the ID:

- In the authorization notification for the payment, with the key `execution.id`
- In the Payrails Portal on the payment’s Payment Details page under `Reference details`, accessed by searching for the payment in the Payments section

<img
  className="mx-auto block rounded-lg object-cover"
  src="/images/docs/orchestration/payment-acceptance/capture-payment-details-reference.png"
  width="80%"
/>

## 2. Make a POST request

Make a `POST` request to the [Capture a payment](https://payrails.readme.io/reference/captureaction) endpoint, where `executionId` is the ID from the previous step.

In your request, include:

- `amount.value`: The decimal amount of the major currency unit, in any precision
- `amount.currency`: ISO 3-letter currency code

For example:

```json
{
  "amount": {
    "value": "12.50",
    "currency": "EUR"
  }
}
```

## 3. Receive the capture response

If the capture was successfully requested, you’ll find the following in the response:

- `actionId`: The unique identifier for this capture execution
- `links`: Links to the next possible actions that can be taken

For the complete response schema, refer to the [Capture a payment](https://payrails.readme.io/reference/captureaction) API reference.

## 4. Wait for the capture notification

Successfully requested payment captures will be processed asynchronously. If you have [notifications](/docs/orchestration/payment-acceptance/receive-notifications) set up, you’ll receive a notification once the processing is complete, informing you whether the capture was a success or failure.

Example notification for a successful capture:

```json
{
  "action": "capture",
  "amount": {
    "currency": "EUR",
    "value": "24.00"
  },
  "execution": {
    "holderReference": "customer123",
    "holderId": "9d113e2a-35a0-40e1-828b-35a18ac35b41",
    "id": "f8c36786-008d-4352-a0cf-b799955dbfc2",
    "merchantReference": "order_3573894940903",
    "workflowCode": "payment-acceptance"
  },
  "paymentComposition": [
    {
      "operationType": "Capture",
      "amount": {
        "currency": "EUR",
        "value": "24.00"
      },
      "integrationType": "api",
      "operationResult": "Success",
      "operationProviderReference": "provider-reference-example",
      "providerId": "5eb08624-ecce-4cbc-9a8a-61d484172815",
      "providerConfigId": "1e308624-e5ce-4cvc-9a8a-61d482172817",
      "paymentId": "c9e1a8ea-a0de-42bd-bcfd-46d3f437b5fe",
      "paymentInstrumentId": "a4a7ed2d-bdcc-46a3-a9f2-701b5d7924b0",
      "storeInstrument": true,
      "paymentMethodCode": "card",
      "success": true,
      "paymentInstrument": {
        "tokens": [
          {
            "type": "psp",
            "reference": "provider-token",
            "meta": {
              "holderReference": "provider-customer-reference"
            }
          }
        ]
      }
    }
  ],
  "success": true
}
```

Example notification for a failed capture:

```json
{
  "action": "capture",
  "actionId": "48c5b5e1-55f9-4ce0-9405-c859285dce1f",
  "amount": {
    "currency": "EUR",
    "value": "0.00"
  },
  "errors": [
    {
      "detail": "Payment can't be captured",
      "id": "e6109321-fd30-46cc-9cdc-5864426c09c3",
      "title": "action.not-allowed"
    }
  ],
  "execution": {
    "holderReference": "customer123",
    "holderId": "9d113e2a-35a0-40e1-828b-35a18ac35b41",
    "id": "f8c36786-008d-4352-a0cf-b799955dbfc2",
    "merchantReference": "order_3573894940903",
    "workflowCode": "payment-acceptance"
  },
  "success": false
}
```

> 📘
>
> Note that a capture will not be possible in all cases, for example if there is no authorized amount for a given execution, or if the payment is already captured. In such cases, refer to `detail` in the `errors` object for an explanation of why the action failed. Below are the most common reasons:

| Error detail                                                                                | Description                                                                                                                  |
| :------------------------------------------------------------------------------------------ | :--------------------------------------------------------------------------------------------------------------------------- |
| `Cannot capture not authorized execution`                                                   | There is no authorized payment related to a given workflow execution.                                                        |
| `Cannot Capture zero amount`                                                                | The total authorized amount is zero and there is no amount that can be captured.                                             |
| `Cannot capture higher amount than authorized`                                              | The amount requested to be captured is greater than the authorized amount.                                                   |
| `Cannot Capture Workflow because amount to capture has different currency than the payment` | The currency sent with the request to capture is different than the currency in which the payment was originally authorized. |
| `Payment is already captured`                                                               | There is no payment related to the execution for which the capture request was made.                                         |

## 5. Fetch the execution status

In addition to waiting for a capture notification, you can fetch the status of an execution at any time via the [Get an execution by ID](ref:getexecution) endpoint, where `executionId` is the ID from step 1 above.

# Capture a payment via the Portal

## 1. Search for the payment

Click on Payments in the Payrails Portal sidebar, and search for the payment you want to capture using any of the following:

- Execution ID
- Merchant reference
- Payrails payment ID
- PSP reference

## 2. Review the Payment Details page

On the Payments page, click on the payment to access its details and the actions that can be taken.

## 3. Capture the payment

Capture the payment by clicking on "Capture" at the top right of the page.

The capture amount will be pre-filled with the authorized amount for the payment. Make a full capture by proceeding with this amount, or change it to a smaller amount for a partial capture.

<img
  className="mx-auto block rounded-lg object-cover"
  src="/images/docs/orchestration/payment-acceptance/capture-payment-portal-dialog.png"
  width="80%"
/>

## 4. Review payment status

You can review the status of the payment by returning to its Payment Details page at any time.

<img
  className="mx-auto block rounded-lg object-cover"
  src="/images/docs/orchestration/payment-acceptance/capture-payment-status-overview.png"
  width="80%"
/>

For further details, refer to the Timeline section of the Payment Details page. API requests, responses, and notifications will be shown here once processed.

<img
  className="mx-auto block rounded-lg object-cover"
  src="/images/docs/orchestration/payment-acceptance/capture-payment-timeline.png"
  width="80%"
/>

If you have [notifications](/docs/orchestration/payment-acceptance/receive-notifications) set up, you’ll also receive a notification once your capture has been processed, informing you whether it was a success or failure.
