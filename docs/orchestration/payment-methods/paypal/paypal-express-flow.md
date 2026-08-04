---
title: "PayPal Express Flow"
---
# PayPal Express
PayPal Express is an accelerated checkout experience that lets buyers quickly complete a purchase using stored PayPal account information. It enables faster checkout flows by retrieving buyer details—such as name, email, and shipping address—directly from PayPal, reducing the need for manual entry and improving conversion.

## Benefits of PayPal Express

* **Streamlined checkout**: Buyer details are pre-filled, avoiding redundant data entry.
* **Improved conversion**: Especially effective on mobile, where users often drop off during form entry.
* **Earlier placement**: Can be embedded on product and cart pages, enabling direct checkout without visiting the full checkout page.
* **Reduced technical complexity**: Shipping address comes from PayPal, minimizing form handling and validation.
* **Supports saved PayPal shipping addresses**: Buyers can reuse previously selected addresses from their PayPal wallet.

## Provider Configuration

Payrails supports two flow types for PayPal, configurable at the provider level. You can change this setting from the integration instance page in the merchant portal for PayPal integrations

| Flow Type   | Description                                                                                                                                                        |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Normal**  | Full checkout flow with buyer redirection to PayPal. Includes standard form steps.                                                                                 |
| **Express** | Accelerated checkout. Buyer is redirected to PayPal early, and shipping address can be collected directly from PayPal and returned via notification from Payrails. |

## Shipping Address Handling

Shipping address behavior is controlled via the `deliveryAddressPreference` field inside order `meta` object in your Payrails integration.

> ℹ️
>
> Note that shipping address can only be collected from PayPal if Express mode as used as described above.

| Value                     | Behavior                                                                                                                                                      |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `"userPreSpecified"`      | Buyer selects from saved PayPal addresses **(Default behavior if no other value is sent)**                                                                    |
| `"noShipping"`            | No address is collected or required (e.g., digital goods)                                                                                                     |
| `"allowProviderOverride"` | Merchant-supplied address is used and updated on PayPal. Note: if you wish for us to send a merchant-provided shipping address, this value must be specified. |

## Notifications

For all PayPal flows, the selected or provided shipping address is returned in the merchant notification payload as in the below example:

```json
"providerResponseAdditionalFields": {
  "purchase_units": [
    {
      "reference_id": "tp_3a8e9555-aa5b-408d-a519-c36e1dd907f1",
      "shipping": {
        "name": {
          "full_name": "John Doe"
        },
        "address": {
          "address_line_1": "Strassburger Strasse",
          "address_line_2": "1",
          "admin_area_2": "Berlin",
          "postal_code": "10405",
          "country_code": "DE"
        }
      }
    }
  ]
}
```
