# Introduction

[Payment Options](doc:payment-options) are the payment methods and instruments available to a customer in a given context. You configure rules for their availability in the Payrails Portal and then use the lookup endpoint to synchronously determine and present the right payment options to customers on the client side.

# Steps

## 1. Get the execution ID

For your request to the lookup endpoint, you’ll need the ID of the workflow execution. You can find the ID in the response to the creation of this workflow execution, with the key `id`

## 2. Make a POST request

Make a `POST` request to the [Lookup payment options](https://payrails.readme.io/reference/lookupaction) endpoint, where `executionId` is the ID from the previous step.

In your request, include:

- `amount.value`: The decimal amount of the major currency unit, in any precision
- `amount.currency`: ISO 3-letter currency code

For example:

```json
{
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
```

For the complete request schema, refer to the [Lookup Payment Options](https://payrails.readme.io/reference/lookupaction) API reference.

> 📘
>
> The payment options available to a customer vary according to context. If the context changes during checkout (e.g. the total amount changes), you may want to look up the payment options again and refresh the results to ensure the payment methods and instruments displayed to your consumer remain accurate.

## 3. Receive the lookup response

If the lookup was successfully executed, you’ll find the following in the response:

- `actionId`: The unique identifier for this capture execution
- `data.paymentCompositionOptions`: This is the list of payment methods and payment instruments that can be used to complete the execution.
- `links`: Links to the next possible actions that can be taken

For example:

```json
{
  "name": "lookup",
  "actionId": "1aa0ef20-36cb-4485-b60e-0526c3699014",
  "executedAt": "2022-02-03T14:38:40.557Z",
  "data": {
    "paymentCompositionOptions": [
      {
        "paymentMethodCode": "card",
        "integrationType": "api",
        "description": "Card.",
        "paymentInstruments": [
          {
            "id": "b8fe6271-5d71-4d28-b8e8-89e64acc0c49",
            "holderId": "788c7c09-a490-4603-915a-e2a957a6cca1",
            "createdAt": "2022-04-22T17:53:36.814Z",
            "paymentMethod": "card",
            "description": "Personal card.",
            "fingerprint": "739abff0-406a-4d32-8cfb-701e12be5848",
            "status": "created",
            "data": {
              "network": "visa",
              "bin": "411111",
              "suffix": "1111",
              "expiryMonth": "10",
              "expiryYear": "2026",
              "holderName": "John Doe"
            },
            "meta": {
              "communityCode": "GAP",
              "userKey": "9890124569"
            }
          }
        ]
      }
    ]
  },
  "links": {
    "execution": "https://api.staging.payrails.io/merchant/workflows/100ade99-ef8d-43de-8a35-4d31dbdb37d0/executions/99e2f33a-2d17-4c98-8242-6bf5a4a08016",
    "authorize": {
      "method": "POST",
      "href": "https://api.staging.payrails.io/merchant/workflows/100ade99-ef8d-43de-8a35-4d31dbdb37d0/executions/99e2f33a-2d17-4c98-8242-6bf5a4a08016/authorize"
    }
  }
}
```

> 📘
>
> By default, the `paymentInstruments` list inside each of the payment methods returned includes only the ones with status `Created` or `Enabled` for the Holder that created the execution. This means that any inactive, expired, or blocked instrument will be excluded to avoid payment rejections.
>
> If you want to query all the instruments for that holder regardless of their current state, you can use the [List Instruments API](/reference/listinstruments).

For the complete response schema, refer to the [Lookup Payment Options](https://payrails.readme.io/reference/lookupaction) API reference.
