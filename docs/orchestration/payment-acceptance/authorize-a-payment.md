# Introduction

After a customer has chosen the payment method and instrument they want to pay with, you’ll need to authorize the payment before you can capture it. Payrails currently supports one payment for each workflow execution but split and multiple payments will be supported soon.

# Requirements

- Have created a workflow execution
- A payment method and instrument must have been provided for the payment
- Have [notifications](https://payrails.readme.io/docs/receive-notifications) set up if you wish to receive them for authorization results

# Steps

## 1. Get the execution ID

For your request to the lookup endpoint, you’ll need the ID of the workflow execution. You can find the ID:

- In response to the creation of this workflow execution, with the key `id`

## 2. Make a POST request

Make a `POST` request to the [Authorize a payment](https://payrails.readme.io/reference/authorizeaction) endpoint, where `executionId` is the ID from the previous step.

In your request, include:

- `amount.value`: The decimal amount of the major currency unit, in any precision
- `amount.currency`: ISO 3-letter currency code
- `returnInfo.success`: URL to redirect customers when the execution stops interacting with them due to progress
- `paymentComposition.integrationType`: The code of the integration type for using the payment method with the provider
- `paymentComposition.amount`: The amount to pay with this payment method or instrument

For example:

```json
{
  "amount": {
    "value": "12.50",
    "currency": "EUR"
  },
  "returnInfo": {
    "success": "https://mysuccessurl.com"
  },
  "paymentComposition": [
    {
      "paymentInstrumentId": "384279fe-fee4-441d-9836-d2ef663551ad",
      "paymentMethodCode": "card",
      "integrationType": "api",
      "amount": {
        "value": "12.50",
        "currency": "EUR"
      }
    }
  ],
  "paymentSplits": [
    {
      "providerAccountId": "BA00000000000000000000003",
      "providerAccountType": "BalanceAccount",
      "providerId": "01039f19-d151-48b4-86aa-b36e68323e4e",
      "providerConfigId": "01039f19-d151-48b4-86aa-b36e68323e4e",
      "amount": {
        "value": "10.50",
        "currency": "EUR"
      }
    },
    {
      "providerAccountId": "COM0000001",
      "providerAccountType": "CommissionAccount",
      "amount": {
        "value": "2.00",
        "currency": "EUR"
      }
    }
  ]
}
```

For the complete request schema, refer to the [Authorize a payment](https://payrails.readme.io/reference/authorizeaction) API Reference.

## 3. Receive the response to the authorization request

If the authorization was successfully requested, you’ll find the following in the response:

- `actionId`: The unique identifier for this capture execution
- `links.execution`: URL at which to fetch the current execution state
- `links.consumerWait`: URL to redirect your customer to until the payment session is complete. This endpoint handles 3DS challenges, APM redirects, and any other PSP-driven authentication step-ups, then returns the customer to your `returnInfo.success` URL when the session resolves. This is the standard way to handle redirects in an API integration.

For example:

```json
{
  "name": "authorize",
  "actionId": "1aa0ef20-36cb-4485-b60e-0526c3699014",
  "executedAt": "2022-04-22T17:53:36.814Z",
  "links": {
    "execution": "https://api.staging.payrails.io/merchant/workflows/100ade99-ef8d-43de-8a35-4d31dbdb37d0/executions/99e2f33a-2d17-4c98-8242-6bf5a4a08016",
    "consumerWait": "https://api.staging.payrails.io/public/redirect/merchant/example-merchant/100ade99-ef8d-43de-8a35-4d31dbdb37d0/99e2f33a-2d17-4c98-8242-6bf5a4a08016/dGVtcG9yYXJ5LWF1dGgK",
    "capture": {
      "method": "POST",
      "href": "https://api.staging.payrails.io/merchant/workflows/100ade99-ef8d-43de-8a35-4d31dbdb37d0/executions/99e2f33a-2d17-4c98-8242-6bf5a4a08016/capture"
    },
    "cancel": {
      "method": "POST",
      "href": "https://api.staging.payrails.io/merchant/workflows/100ade99-ef8d-43de-8a35-4d31dbdb37d0/executions/99e2f33a-2d17-4c98-8242-6bf5a4a08016/cancel"
    }
  }
}
```

For the complete response schema, refer to the [Authorize a payment](https://payrails.readme.io/reference/authorizeaction) API Reference.

<Note>
  Handling 3DS yourself? If you want full control over the 3DS redirect (for example, to avoid showing a Payrails-hosted page on non-3DS payments), see [3D Secure](./3d-secure)  for the manual integration pattern using `links.3ds`.
</Note>

## 4. Wait for authorization notification

Successfully requested payment authorizations will be processed asynchronously. If you have notifications 🔗 set up, you’ll receive a notification once the processing is complete, informing you whether the authorization was a success or failure.

Example notification for a successful authorization:

```json
{
  "action": "authorize",
  "actionId": "b0c28520-eca0-4be6-8771-99c43144bdaf",
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
      "operationType": "Authorize",
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
      "threeDS": {
        "authenticationValue": "AAABAVIREQAAAAAAAAAAAAAAAAA=",
        "challenged": true,
        "dsTransId": "5ed5d1d0-982f-45f4-97a1-68651ac429d0",
        "eci": "05",
        "enrolled": "Y",
        "exemptionApplied": "none",
        "exemptionIndicator": "none",
        "transStatus": "Y",
        "version": "2.2.0"
      },
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

Example notification for a failed authorization:

```json
{
  "action": "authorize",
  "actionId": "d31d96a5-979b-4f0b-bc31-a20e820e1816",
  "amount": {
    "currency": "EUR",
    "value": "24.00"
  },
  "errors": [
    {
      "detail": "The payment method produced a non-successful status.",
      "id": "4a8fe381-6a77-4810-b72d-4cc00e5606c5",
      "title": "execution.payment-method.failed"
    }
  ],
  "execution": {
    "holderReference": "customer123",
    "holderId": "9d113e2a-35a0-40e1-828b-35a18ac35b41",
    "id": "f8c36786-008d-4352-a0cf-b799955dbfc2",
    "merchantReference": "order_3573894940903",
    "workflowCode": "payment-acceptance"
  },
  "paymentComposition": [
    {
      "errors": [
        {
          "detail": "The payment method produced a non-successful status.",
          "id": "4a8fe381-6a77-4810-b72d-4cc00e5606c5",
          "title": "execution.payment-method.failed"
        }
      ],
      "operationResult": "InsufficientBalance",
      "operationType": "Authorize",
      "success": false,
      "amount": {
        "currency": "EUR",
        "value": "24.00"
      },
      "integrationType": "api",
      "operationProviderReference": "provider-reference-example",
      "providerId": "5eb08624-ecce-4cbc-9a8a-61d484172815",
      "providerConfigId": "1e308624-e5ce-4cvc-9a8a-61d482172817",
      "paymentId": "c9e1a8ea-a0de-42bd-bcfd-46d3f437b5fe",
      "paymentInstrumentId": "a4a7ed2d-bdcc-46a3-a9f2-701b5d7924b0",
      "storeInstrument": true,
      "paymentMethodCode": "card",
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
  "success": false
}
```

For more information on reasons why an authorization may fail, refer to the [Result Codes](/docs/resources/payments/operation-results) page.

## 5. Fetch the execution status

In addition to waiting for an authorization notification, you can fetch the status of an execution at any time via the [Get an execution by ID](/reference/getexecution) endpoint, where `executionId` is the ID from step 1.

For long-running operations, you can use the `waitWhile[status]` query parameter to long-poll the endpoint:

```http
GET /merchant/workflows/{workflowCode}/executions/{executionId}?waitWhile[status]=authorizeRequested
```

The request blocks until the execution moves out of the specified status. This is the most resilient mechanism for fetching execution updates and is recommended over short-polling.
