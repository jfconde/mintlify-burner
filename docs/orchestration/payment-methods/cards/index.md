# Introduction
Accepting card payments is essential for any online business, as it allows customers to make transactions quickly and securely. Payrails simplifies the process of accepting card payments, providing a seamless experience for both you and your customers. This guide will walk you through the steps to integrate card payments into your app or website using Payrails, ensuring a smooth and reliable transaction process.

# Pre-requisites

Before you start accepting card payments with Payrails, there are a few requirements you must meet:

1. Integrate with Payrails using one of our [SDKs](/docs/orchestration/checkout-sdks/index) or our [API](/docs/orchestration/checkout-sdks/accept-payments-via-api)
2. Configure a new [integration account](/docs/orchestration/integrations/index) for cards via one of the following Payment Service Providers:  
   [Adyen](/docs/orchestration/integrations/adyen), [AmEx](/docs/orchestration/integrations/amex), [Checkout.com](/docs/orchestration/integrations/checkout-com), [dLocal](/docs/orchestration/integrations/dlocal), [Fawry](/docs/orchestration/payment-methods/fawrypay), [Flutterwave](/docs/orchestration/integrations/index), [Garanti BBVA](/docs/orchestration/integrations/index), [HyperPay](/docs/orchestration/integrations/index),  [Lidio](/docs/orchestration/integrations/index), [PayU](/docs/orchestration/integrations/index), [Pine Labs](/docs/orchestration/integrations/index), [Stripe](/docs/orchestration/integrations/stripe), [Tap](/docs/orchestration/integrations/index), [Unlimit](/docs/orchestration/integrations/index) and _coming soon (Nuvei, WorldPay)_
3. Enable cards as a [payment option](/docs/orchestration/payment-methods/set-up-payment-methods).
4. Make sure you're sending the provider specific card meta fields (found under Merchant configurations -> Meta fields on your Payrails dashboard and in the examples below) in your requests.

# Ways to integrate cards

Payrails offers two main integration types for accepting card payments, each with its own set of advantages:

## Payrails SDK

Payrails provides a hosted payment page, which is a fully customizable and responsive payment form hosted on our secure servers. This option is perfect for businesses that want to offer a seamless payment experience without the need to handle sensitive card information directly.

#### Benefits:

* Reduces your PCI-DSS compliance scope.
* Provides a customizable and responsive payment form.
* Minimizes the need for complex front-end development.

#### Integration Steps:

1. Configure the hosted payment page settings in your Payrails dashboard.
2. Redirect customers to the hosted payment page URL when they initiate a payment.
3. Handle the response from Payrails after the transaction is completed.

## Server-to-server integration

Payrails also provides an API integration option, which allows you to build a fully custom payment form within your app or website. This option requires a higher level of PCI-DSS compliance but offers more flexibility and control over the payment experience.

#### Benefits:

* Complete control over the payment form's design and functionality.
* Enables seamless integration with your existing user interface.
* Provides more flexibility in handling payment-related events and actions.

#### Integration Steps:

1. Collect card information from the customer using a secure payment form on your app or website.
2. Tokenize the card data using the Payrails API, which securely sends the data to Payrails and returns a token.
3. Use the token to process the payment through the Payrails API, specifying the desired PSP and transaction details.
4. Handle the response from Payrails to confirm the transaction's success or handle any errors.

Choose the integration type that best suits your business needs and follow the appropriate steps to start accepting card payments with Payrails.

#### Parse card from lookup response

With a server-to-server integration, you can call our [lookup payment options](/docs/orchestration/payment-acceptance/lookup-payment-options) endpoint to get available payment options and relevant configurations for each payment method. As the example below, you can see `genericRedirect` returned as an option of the `paymentCompositionOptions`. You can use this value later to authorize payments with Payrails as you can see in the next sections.

```json
{
    "name": "lookup",
    "actionId": "0bb6413e-cabb-4074-99e6-9e815c69f25b",
    "executedAt": "2024-05-08T12:33:21.527395295Z",
    "data": {
        "paymentCompositionOptions": [
            {
                "integrationType": "api",
                "paymentMethodCode": "card",
                "description": "Card"
            }
        ]
    },
    "links": {
        "execution": "http://payrails-api.staging.payrails.io/merchant/workflows/payment-acceptance/executions/83c534ac-13b7-43e6-b04b-f3e8b4eb4424",
        "authorize": {
            "method": "POST",
            "href": "http://payrails-api.staging.payrails.io/merchant/workflows/payment-acceptance/executions/83c534ac-13b7-43e6-b04b-f3e8b4eb4424/authorize"
        }
    }
}
```

#### Pass card payment method in request to authorize payment with Payrails

You can then make a request to our [authorize a payment endpoint](/reference/authorizeaction) with `card` as the `paymentMethodCode`. See an example below:

```json
{
  "executionId": "c0fd1c51-e709-47e5-bfd1-5d1c98f7d990",
  "amount": {
    "value": "100",
    "currency": "USD"
  },
  "paymentComposition": [{
    "integrationType": "api",
    "paymentMethodCode": "card",
    "amount": {
      "value": "100",
      "currency": "USD"
    }
  }],
  "meta": {
    "order": {
      "lines": [{
        "id": "UUID",
        "name": "Order Name",
        "quantity": 1,
        "unitPrice": {
          "currency": "USD",
          "value": "100"
        }
      }]
    }
  },
  "returnInfo": {
    "success": "https://mysuccessurl.com",
    "error": "https://myerrorurl.com"
  }
}
```

# Supported regions / countries

<Table align={["left","left"]}>
  <thead>
    <tr>
      <th>
        Region(s)
      </th>

      <th>
        Countries
      </th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td>
        Europe
      </td>

      <td>
        🇦🇹 Austria, 🇧🇪 Belgium, 🇧🇬 Bulgaria, 🇭🇷 Croatia, 🇨🇾 Cyprus,
        🇨🇿 Czech Republic, 🇩🇰 Denmark, 🇪🇪 Estonia, 🇫🇮 Finland,
        🇫🇷 France, 🇩🇪 Germany, :gibraltar: Gibraltar, 🇬🇷 Greece, 🇭🇺 Hungary,
        🇮🇸 Iceland, 🇮🇪 Ireland, 🇮🇹 Italy, 🇱🇻 Latvia, 🇱🇮 Liechtenstein,
        🇱🇹 Lithuania, 🇱🇺 Luxembourg, 🇲🇹 Malta, 🇳🇱 Netherlands, 🇳🇴 Norway,
        🇵🇱 Poland, 🇵🇹 Portugal, 🇷🇴 Romania, 🇸🇰 Slovakia, 🇸🇮 Slovenia,
        🇪🇸 Spain, 🇸🇪 Sweden, 🇨🇭 Switzerland, :tr: Turkey, :gb: UK, others..
      </td>
    </tr>

    <tr>
      <td>

      </td>

      <td>

      </td>
    </tr>

    <tr>
      <td>

      </td>

      <td>

      </td>
    </tr>

    <tr>
      <td>
        North and Central America
      </td>

      <td>
        🇺🇸 USA, 🇲🇽 Mexico, :canada: Canada, 🇨🇷 Costa Rica, 🇸🇻 El Salvador,
        🇬🇹 Guatemala, 🇭🇳 Honduras, :nicaragua: Nicaragua, 🇵🇦 Panama,
        :dominican_republic: Dominican Republic, :puerto_rico: Puerto Rico, others..
      </td>
    </tr>

    <tr>
      <td>
        South America
      </td>

      <td>
        🇦🇷 Argentina, 🇧🇷 Brazil, :bolivia: Bolivia, 🇨🇱 Chile, 🇨🇴 Colombia,
        :ecuador: Ecuador, :paraguay: Paraguay, 🇵🇪 Peru, :uruguay: Uruguay, others..
      </td>
    </tr>

    <tr>
      <td>
        Africa
      </td>

      <td>
        :cameroon: Cameroon, 🇪🇬 Egypt, :ghana: Ghana, :kenya: Kenya, :morocco: Morocco,
        :malawi: Malawi, 🇳🇬 Nigeria, :rwanda: Rwanda, :senegal: Senegal, 🇿🇦 South Africa,
        :tanzania: Tanzania, :uganda: Uganda, others..
      </td>
    </tr>

    <tr>
      <td>
        Asia Pacific
      </td>

      <td>
        :australia: Australia, :bahrain: Bahrain, :bangladesh: Bangladesh, :cn: China, :hong_kong: Hong Kong,  
        :india: India, :indonesia: Indonesia, :iraq: Iraq, :israel: Israel, :jp: Japan, :jordan: Jordan,
        :kuwait: Kuwait, :malaysia: Malaysia, :maldives: Maldives, :new_zealand: New Zealand, :oman: Oman,
        :pakistan: Pakistan, :philippines: Philippines, :saudi_arabia: Saudi Arabia, :singapore: Singapore, :kr: South Korea,
        :sri_lanka: Sri Lanka, :taiwan: Taiwan, :thailand: Thailand, :united_arab_emirates: UAE, :vietnam: Vietnam, others..
      </td>
    </tr>
  </tbody>
</Table>

# Supported workflows and services

| Workflow                                                                                         | Supported                                |
| :----------------------------------------------------------------------------------------------- | :--------------------------------------- |
| Available via Payrails SDK                                                                       | :heavy_check_mark:                       |
| Available via Payrails API                                                                       | :heavy_check_mark:                       |
| [Delayed / Manual Capture](/docs/orchestration/payment-acceptance/capture-and-cancel-modes#capture-mode) | :heavy_check_mark:                       |
| [Instant Capture](/docs/orchestration/payment-acceptance/capture-and-cancel-modes#capture-mode)          | :heavy_check_mark:                       |
| [Cancel / Void](/docs/orchestration/payment-acceptance/cancel-a-payment)                                 | ✔️                                       |
| [Refund / Reverse](/docs/orchestration/payment-acceptance/refund-a-payment)                              | :heavy_check_mark:                       |
| [Save Instruments](/docs/token-vault/tokenize-payment-instruments/index)                  | :heavy_check_mark:                       |
| [Merchant Initiated Transaction (MIT)](/docs/resources/payments/authorization-flags)       | :heavy_check_mark:                       |
| Interoperability                                                                                 | Vault tokens, PSP tokens, Network tokens |
