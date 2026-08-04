---
description: This page describes the way that you can capture or cancel your payments with Payrails.
---

When authorizing a payment, you can choose to do it in two ways depending on your case:

- **One-step**: Authorize and Capture in the same step.
- **Two-Step**: Authorize first, and then capture in a different step.

Both options have pros and cons, and which one you use will depend on your type of business, use case, and factors like how often you change the authorized amount before actually capturing it.

The availability of each flow will depend on the providers you choose for processing the payment and other factors (payment method, type of card, country, etc.).

Good news! Payrails supports both modes, and it’s easy to configure your workflows to choose one or the other by default. Also, if for a specific workflow execution you need to override the default behavior, you can also do it.

## Capture Mode

| `captureMode`      | Description                                                                                                                                                                                                                                                                                                                                                                    | Recommended if…                                                                                                                   |
| :----------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------- |
| `Instant`          | Authorize and capture payments in one step. If the payment provider has a specific endpoint for this type of operation (usually called “sale” or “charge”), we will use it. Otherwise, we will authorize and immediately capture your payment. Regardless of how many calls we make to the provider, you will only get one notification for this operation when it’s captured. | You don’t want to separate the authorization and capture steps in your process.                                                   |
| `Manual` (default) | In a two-step authorization, Payrails authorizes the payment but doesn’t capture neither schedule an automatic capture. For capturing the payment, you will have to execute the [Capture action](ref:captureaction).                                                                                                                                                           | You want to have full control of the capture. Or have cases in which the capture amount is different than the authorized.         |
| `Delayed`          | In a two-step authorization, Payrails authorizes the payment and schedules its capture for the time indicated in the `captureDelay` field. If the payment is [Captured](ref:captureaction) or [Canceled](ref:cancelaction) before that time, the schedule is discarded.                                                                                                        | You want to have a buffer time between authorizing and capturing your payments, but don’t want to execute the `Capture` yourself. |

## Cancel Mode

> 📘 Cancel vs. Refund vs. Void
>
> In this case, we use the term “Cancel” to refer to canceling an authorization of a payment that has not been captured yet.
>
> A Refund can only be performed after a payment is Captured.

| `cancelMode`       | Description                                                                                                                                                                                                                                                        | Recommended if…                                                                                       |
| :----------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------- |
| `Manual` (default) | In a two-step authorization, Payrails authorizes the payment but doesn’t capture, cancel, or schedule any of them. This behaviour will leave up to the bank to release the funds eventually (usually up to 7 days), which may create friction with your customers. | You want full control of your payment process and don’t care about releasing the funds automatically. |
| `Delayed`          | In a two-step authorization, Payrails authorizes the payment and schedules its canceling in case it’s neither [Captured](ref:captureaction) or [Canceled](ref:cancelaction) by you before. The delay time is determined by the `cancelDelay` field.                | You want to release the funds automatically in case a capture isn’t made on time.                     |

> ❗️ Delays in both are not allowed
>
> Please note that there could be a possible conflict if you set the value `Delayed` to both `captureMode` and `cancelMode`. We will reject that configuration because it can lead to unexpected behaviours.

## Delay defaults and formats

For both `captureDelay` and `cancelDelay`, we use the [ISO 8601 standard](https://tc39.es/proposal-temporal/docs/duration.html). If it’s your first time using it, you may find it a bit strange, but it’s super powerful!

If you choose a captureMode or cancelDelay of type `Delayed` but you don’t indicate a delay time in their respective fields, our **default is 5 minutes**.

## Multiple captures

By default, all our captures are final in the payment processor. If you want a different behaviour, please indicate it to your integration team.

This means, for example, that if you authorized 10 EUR, and then capture 7 EUR, the remaining 3 EUR are immediately released.
