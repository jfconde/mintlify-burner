# Introduction

All operations in Payrails are defined and executed with customized [Workflows](/docs/overview/workflows), which are lists of steps to be performed in sequence. The `payment-acceptance` workflow, for example, is a list of steps for accepting payments, and you must create an execution of this workflow via the Payrails API before initiating a payment.

After creating an execution, you can [Lookup payment options](https://payrails.readme.io/reference/lookupaction) to present to your customer.

# Steps

## 1. Make a POST request

Make a `POST` request to the [Create a workflow execution](/reference/createexecution) endpoint.

In your request, make sure to include:

- `merchantReference`: The merchant-provided reference for the execution. This is commonly the identifier of the order in the merchant’s system.

The subsequent actions (refer to Step 3 below) of starting a payment session or looking up payment options can also be included in your request at this time, as part of the `initialActions`.

For example:

```json
{
  "initialActions": [
    {
      "action": "lookup",
      "method": "POST",
      "body": {
        "amount": {
          "value": "12.50",
          "currency": "EUR"
        },
        "meta": {
          "country": {
            "code": "DE"
          },
          "customer": {
            "reference": "customer123",
            "country": {
              "code": "DE"
            }
          },
          "clientContext": {
            "osType": "ios",
            "language": "de-DE"
          },
          "allowedPaymentMethods": ["card", "klarna"]
        }
      }
    }
  ],
  "merchantReference": "order_3573894940903",
  "holderReference": "1231905323475",
  "meta": {
    "order": {
      "reference": "order_3573894940903"
    },
    "customer": {
      "reference": "1231905323475",
      "country": {
        "code": "DE"
      }
    },
    "clientContext": {
      "ipAddress": "217.110.239.132",
      "osType": "ios",
      "origin": "https://example.com"
    },
    "allowNative3DS": false
  }
}
```

For the complete request schema, refer to the [Create an execution](https://payrails.readme.io/reference/createexecution) API Reference.

## 2. Receive response

The API response for creating an execution will include the following:

- `id`: The unique identifier for this execution
- `status`: Business-case dependent set of status tags of this execution.

For the complete response schema, refer to the [Create an execution](https://payrails.readme.io/reference/createexecution) API Reference.

## 3. Continue with the payment lifecycle

Once you’ve created a workflow execution, you can proceed with the payment in two ways:

- If you know the payment method and/or instrument that should be used (because you included the lookup into the `initialActions` or because it is set by your configurations (e.g. default set by customer):
  - [Authorize a payment](ref:authorizeaction) with the selected payment method and instrument.
- [Lookup payment options](https://payrails.readme.io/reference/lookupaction)
  - This is for cases in which you’ll present payment options to your customer based on rules you’ve configured in the Payrails Portal, and authorize the payment after they’ve selected a payment option and provided an instrument.
- Important: if you have meta fields specified in **Payment options** configuration in the portal, they must be included in the request.

After performing an authorization, you’ll be able to **capture**, **cancel**, **refund**, or **confirm** the payment.
