---
description: "PayPal is an alternative payment method around the globe. With PayPal, customers can enjoy a seamless checkout experience, with a choice of fulfilling their payments with their wallet balance or any payment method that is linked to their account, e.g saved cards, bank accounts, e-wallets, etc.. Via PayPal, customers can also use a BNPL payment scheme and install the purchase amount directly to PayPal."
title: "PayPal"
---
# Introduction
PayPal is a global online payment system that allows users to pay for goods and services, send money, and accept payments without revealing their financial details. It's widely accepted around the world and used by millions of businesses and individuals daily.

With Payrails' direct PayPal integration you can accept payments, create billing agreements with PayPal and charge customers for recurring payments, and implement PayPal either using our SDKs or with a server-to-server integration.

# Pre-requisites

In order to use Payrails' PayPal integration, you need to grant us permission to make requests on behalf of your PayPal merchant account. In order to initiate this process, please contact your Payrails account representative. We will send you a link to a PayPal onboarding page where you can grant us access to make calls on your behalf. Please note that you have to separately connect your sandbox PayPal account and your production account.

Before you start accepting PayPal payments with Payrails, there are a few requirements you must meet:

1. Integrate with Payrails using one of our [SDKs](/docs/orchestration/checkout-sdks/index) or our [API](/docs/orchestration/checkout-sdks/accept-payments-via-api)
2. Configure a new [integration account](/docs/orchestration/integrations/index) for PayPal via [PayPal](/docs/orchestration/integrations/paypal)
3. Enable PayPal as a [payment option](/docs/orchestration/payment-methods/set-up-payment-methods).
4. Make sure you're sending the PayPal specific meta fields (found under Merchant configurations -> Meta fields on your Payrails dashboard and in the examples below) in your requests.

# Ways to integrate

## Payrails SDK

The simplest way to use PayPal with Payrails is to use our [drop-in](/docs/orchestration/checkout-sdks/web-v5-legacy/drop-in) in your checkout flow. With this integration type, no additional development work is required to accept payments with PayPal.

## Server-to-server integration

To use PayPal by completely managing your own client-side implementation, and using Payrails APIs with a server-to-server integration to process payments with PayPal. With this approach, follow the documentation below to build PayPal into your applications:

#### Parse PayPal from lookup response

With a server-to-server integration, you can call our [lookup payment options](/docs/orchestration/payment-acceptance/lookup-payment-options) endpoint to get available payment options and relevant configurations for each payment method. As the example below, you can see `payPal` returned as an option of the `paymentCompositionOptions`. You can use this value later to authorize payments with Payrails as you can see in the next sections.

```json
{
    "name": "lookup",
    "actionId": "0bb6413e-cabb-4074-99e6-9e815c69f25b",
    "executedAt": "2024-05-08T12:33:21.527395295Z",
    "data": {
        "paymentCompositionOptions": [
            {
                "integrationType": "api",
                "paymentMethodCode": "payPal",
                "description": "PayPal"
            }
        ]
    },
    "links": {
        "execution": "http://payrails-api.staging.payrails.io/merchant/workflows/payment-acceptance/executions/c0fd1c51-e709-47e5-bfd1-5d1c98f7d990",
        "authorize": {
            "method": "POST",
            "href": "http://payrails-api.staging.payrails.io/merchant/workflows/payment-acceptance/executions/c0fd1c51-e709-47e5-bfd1-5d1c98f7d990/authorize"
        }
    }
}
```

#### Pass PayPal payment method in request to authorize payment with Payrails

You can then make a request to our [Authorize a payment endpoint](/reference/authorizeaction) with `payPal` as the `paymentMethodCode`. See an example below:

```json
{
  "executionId": "c0fd1c51-e709-47e5-bfd1-5d1c98f7d990",
  "amount": {
    "value": "100",
    "currency": "USD"
  },
  "paymentComposition": [{
    "integrationType": "api",
    "paymentMethodCode": "payPal",
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

#### Completing the Payment Flow

After initiating the authorization, use the [Get an execution by ID endpoint](/reference/getexecution) to fetch the execution details. In the API response, locate the `requiredAction` object (see example below). If present, this indicates that user interaction is required. Extract the `requiredAction.href` URL from this object.

```json
{
    "id": "c0fd1c51-e709-47e5-bfd1-5d1c98f7d990",
    "status": [
        {
            "code": "authorizePending",
            "time": "2024-05-08T12:33:25.362443Z"
        },
        {
            "code": "created",
            "time": "2024-05-08T12:33:21.758224Z"
        },
        {
            "code": "authorizeRequested",
            "time": "2024-05-08T12:33:23.401641Z"
        }
    ],
    "createdAt": "2024-05-08T12:33:21.758224Z",
    "merchantReference": "a85d6211-26d0-4a12-ae85-4d576a700922",
    "holderId": "5fc2d688-c19c-45c5-9c38-57072fcda416",
    "workspaceId": "7f9f1882-a103-408d-ac96-46a7021e537a",
    "holderReference": "9d42ef80-3aa4-4485-895e-4207bc97a966",
    "amount": {
        "value": "100.00",
        "currency": "USD"
    },
    "paymentComposition": [
        {
            "integrationType": "api",
            "paymentMethodCode": "payPal",
            "amount": {
                "value": "100.00",
                "currency": "USD"
            }
        }
    ],
    "workflow": {
        "code": "payment-acceptance",
        "version": 4
    },
    "meta": {
        "CIT": true,
        "order": {
            "lines": [
                {
                    "id": "c5f19f65-48d5-4449-8d60-b9c01547ca8e",
                    "name": "Order Name",
                    "quantity": 1,
                    "unitPrice": {
                        "currency": "USD",
                        "value": "100"
                    }
                }
            ]
        },
        "risk": {
            "sessionId": "69eb6092-9fbd-47bb-8016-7360f7786215"
        }
    },
    "returnInfo": {
        "success": "https://mysuccessurl.com",
        "error": "https://myerrorurl.com"
    },
    "links": {
        "self": "http://payrails-api.staging.payrails.io/merchant/workflows/payment-acceptance/executions/c0fd1c51-e709-47e5-bfd1-5d1c98f7d990",
        "redirect": "http://payrails-api.staging.payrails.io/public/redirect/merchant/payment-acceptance/c0fd1c51-e709-47e5-bfd1-5d1c98f7d990/ixBNaQZD5v5FqfswOlKRqOD0QpIqBIsbqB_s5yGMy_OfAhF7MDAwMA==",
        "confirm": {
            "method": "POST",
            "href": "http://payrails-api.staging.payrails.io/merchant/workflows/payment-acceptance/executions/c0fd1c51-e709-47e5-bfd1-5d1c98f7d990/confirm",
            "action": {
                "type": "confirm",
                "redirectUrl": "https://www.sandbox.paypal.com/agreements/approve?ba_token=BA-2RN64831HG685960Y",
                "redirectMethod": "GET",
                "parameters": {
                    "tokenId": "BA-2RN64831HG685960Y"
                }
            }
        }
    },
    "actionRequired": "redirect",
    "requiredAction": {
        "method": "GET",
        "href": "http://payrails-api.staging.payrails.io/public/redirect/merchant/payment-acceptance/c0fd1c51-e709-47e5-bfd1-5d1c98f7d990/ixBNaQZD5v5FqfswOlKRqOD0QpIqBIsbqB_s5yGMy_OfAhF7MDAwMA==",
        "type": "redirect",
        "subType": "client"
    }
}
```

Redirect the customer to the `requiredAction.href` URL. They will be taken to the PayPal login page to complete the payment. After the customer completes the payment, Payrails will send a server-to-server notification to your configured webhook endpoint. This payload will contain the `holderReference`and `paymentInstrumentId` required for performing future recurring (MIT) payments.

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
        Americas
      </td>

      <td>
        :anguilla: Anguilla, :antigua_barbuda: Antigua and Barbuda, :argentina: Argentina, :aruba: Aruba, :bahamas: Bahamas,
        :barbados: Barbados, :belize: Belize, :bermuda: Bermuda, :bolivia: Bolivia, :brazil: Brazil, :british_virgin_islands: British Virgin Islands,
        :canada: Canada, :cayman_islands: Cayman Islands, :chile: Chile, :colombia: Colombia, :costa_rica: Costa Rica, :dominica: Dominica,
        :dominican_republic: Dominican Republic, :ecuador: Ecuador, :el_salvador: El Salvador, :falkland_islands: Falkalnd Islands, :greenland: Greenland,
        :grenada: Grenada, :guatemala: Guatemala, :guyana: Guyana, :honduras: Honduras, :jamaica: Jamaica, :mexico: Mexico,
        :montserrat: Montserrat, Netherlands Antilles, :nicaragua: Nicaragua, :panama: Panama, :paraguay: Paraguay, :peru: Peru,
        :st_kitts_nevis: St Kitts & Nevis, :st_lucia: St Lucia, :st_pierre_miquelon: St Pierre & Miquelon, :st_vincent_grenadines: St Vincent & Grenadines,
        :suriname: Suriname, :trinidad_tobago: Trinidad & Tobago, :turks_caicos_islands: Turks and Caicos, :us: US, :uruguay: Uruguay, :venezuela: Venezuela
      </td>
    </tr>

    <tr>
      <td>
        Asia Pacific
      </td>

      <td>
        :armenia: Armenia, :australia: Australia, :bahrain: Bahrain, :bhutan: Bhutan, :brunei: Brunei, :cambodia: Cambodia, :cn: China,  
        :cook_islands: Cook Islands, :fiji: Fiji, :french_polynesia: French Polynesia, :hong_kong: Hong Kong, :india: India, :israel: Israel,
        :indonesia: Indonesia, :jp: Japan, :jordan: Jordan, :kazakhstan: Kazakhstan, :kiribati: Kiribati, :kuwait: Kuwait, :kyrgyzstan: Kyrgyzstan,
        :laos: Laos, :malaysia: Malaysia, :maldives: Maldives, :marshall_islands: Marshall Islands, :micronesia: Micronesia, :mongolia: Mongolia,
        :nauru: Nauru, :nepal: Nepal, :new_caledonia: New Caledonia, :new_zealand: New Zealand, :niue: Niue, :norfolk_island: Norfolk Islands,
        :oman: Oman, :palau: Palau, :papua_new_guinea: Papua New Guinea, :philippines: Philippines, :pitcairn_islands: Pitcairn Islands, :qatar: Qatar,
        :samoa: Samoa, :saudi_arabia: Saudi Arabia, :singapore: Singapore, :solomon_islands: Solomon Islands, :kr: South Korea,
        :sri_lanka: Sri Lanka, :taiwan: Taiwan, :tajikistan: Tajikstan, :thailand: Thailand, :tonga: Tonga, :turkmenistan: Turkmenistan, :tuvalu: Tuvalu,
        :united_arab_emirates: UAE, :vanuatu: Vanuatu, :vietnam: Vietnam, :wallis_futuna: Wallis & Futuna, :yemen: Yemen
      </td>
    </tr>

    <tr>
      <td>
        Africa
      </td>

      <td>
        :algeria: Algeria, :angola: Angola, :benin: Benin, :botswana: Botswana, :burkina_faso: Burkina Faso, :burundi: Burundi, :cameroon: Cameroon,  
        :cape_verde: Cape Verde, :chad: Chad, :comoros: Comoros, :cote_divoire: Ivory Coast, :congo_kinshasa: DR Congo, :djibouti: Djibouti, :egypt: Egypt,
        :eritrea: Eriteria, :ethiopia: Ethiopia, :gabon: Gabon, :gambia: Gambia, :guinea: Guinea, :guinea_bissau: Guinea-Bissau, :kenya: Kenya,
        :lesotho: Lesotho, :madagascar: Madagascar, :malawi: Malawi, :mali: Mali, :mauritania: Mauritania, :mauritius: Mauritius, :morocco: Morocco,
        :mozambique: Mozambique, :namibia: Namibia, :niger: Niger, :nigeria: Nigeria, :congo_brazzaville: Congo, :rwanda: Rwanda, :st_helena: St Helena,
        :sao_tome_principe: Sao Tome and Principe, :senegal: Senegal, :seychelles: Seychelles, :sierra_leone: Sierra Leone, :somalia: Somalia,
        :south_africa: South Africa, :swaziland: Swaziland, :tanzania: Tanzania, :togo: Togo, :tunisia: Tunisia, :uganda: Uganda, :zambia: Zambia,
        :zimbabwe: Zimbabwe
      </td>
    </tr>

    <tr>
      <td>
        Europe
      </td>

      <td>
        :albania: Albania, :andorra: Andorra, :austria: Austria, :azerbaijan: Azerbaijan, :belarus: Belarus, :belgium: Belgium, :bulgaria: Bulgaria,  
        :bosnia_herzegovina: Bosnia and Herzegovina, :croatia: Croatia, :cyprus: Cyprus, :czech_republic: Czech Republic, :denmark: Denmark,
        :estonia: Estonia, :faroe_islands: Faroe Islands, :finland: Finland, :fr: France, :georgia: Georgia, :de: Germany, :greece: Greece,
        :hungary: Hungary, :iceland: Iceland, :ireland: Ireland, :it: Italy, :latvia: Latvia, :liechtenstein: Liechtenstein, :lithuania: Lithuania,
        :luxembourg: Luxembourg, :macedonia: Macedonia, :malta: Malta, :moldova: Moldova, :monaco: Monaco, :montenegro: Montenegro,
        :netherlands: Netherlands, :norway: Norway, :poland: Poland, :portugal: Portugal, :romania: Romania, :ru: Russia,
        :san_marino: San Marino, :serbia: Serbia, :slovakia: Slovakia, :slovenia: Slovenia, :es: Spain, :svalbard_jan_mayen: Svalbard & Jan Mayen,
        :sweden: Sweden, :switzerland: Switzerland, :ukraine: Ukraine, :gb: UK, :vatican_city: Vatican City
      </td>
    </tr>
  </tbody>
</Table>

# Supported workflows and services

| Workflow                                                                                         | Supported                |
| :----------------------------------------------------------------------------------------------- | :----------------------- |
| Available via Payrails SDK                                                                       | :heavy_check_mark:       |
| Available via Payrails API                                                                       | :heavy_check_mark:       |
| [Delayed / Manual Capture](/docs/orchestration/payment-acceptance/capture-and-cancel-modes#capture-mode) | :heavy_multiplication_x: |
| [Instant Capture](/docs/orchestration/payment-acceptance/capture-and-cancel-modes#capture-mode)          | :heavy_check_mark:       |
| [Cancel / Void](/docs/orchestration/payment-acceptance/cancel-a-payment)                                 | :heavy_check_mark:       |
| [Refund / Reverse](/docs/orchestration/payment-acceptance/refund-a-payment)                              | :heavy_check_mark:       |
| [Save Instruments](/docs/token-vault/tokenize-payment-instruments/index)                  | :heavy_check_mark:       |
| [Merchant Initiated Transaction (MIT)](/docs/resources/payments/authorization-flags)       | :heavy_check_mark:       |
| Interoperability                                                                                 | PSP tokens               |

# PayPal billing agreements

* Fraud SDKs
