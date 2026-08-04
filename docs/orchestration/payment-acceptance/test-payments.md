# Introduction

Testing is critical during the integration phase while you are developing your integration, discovering Payrails' payment processing features, or ensuring your configurations are working as expected.

Payrails provides a testing feature in the Portal to simulate real-life scenarios by creating test payments. It enables:

1. **Testing provider configurations:** You can test the configuration for each provider to ensure it is working correctly.
2. **Simulating dynamic payment options:** Depending on your configurations, you can test which payment options are displayed to your customers on your checkout page.
3. **Simulating dynamic routing:** You can test which provider is selected to route the payment according to your routing configurations.
4. **Testing payment scenarios:** You can test payments to test scenarios such as authorizing by card brand or country, failure scenarios such as declines, fraud, invalid data, or payer authentication with 3D Secure.

# How does it work?

- Login to the Portal and go to the **Test Payments** page.
- The system will generate a unique merchant reference and holder reference each time when you open the page so that you can investigate further details of the payment afterward.
- You can also set these reference values yourself by replacing the auto-generated values. This is useful when using the same holder reference for multiple payments or listing a particular holder's saved instruments.
- You can add meta fields to your payment context to simulate dynamic payment options or routing configurations.
- After you input the fields, you can start the test. It will load the Payrails Drop-in on the page, allowing you to list available payment methods and saved instruments, pay with configured alternative payment methods, or manually enter a new card, optionally saving it for later.
- After you select the payment method to pay, the payment will be completed and you will be able to see the result and details of the payment on the Executions and Payments pages.

<img
  className="mx-auto block rounded-lg object-cover"
  src="/images/docs/orchestration/payment-acceptance/test-payments-page-overview.png"
  width="80%"
/>

# Test PSP

To further increase your testing capabilities and simplify your integration process, we developed a **Test PSP**. This "fake" or "mock" payment provider helps you simulate the entire payment flow without using an external provider.

This provider is enabled by default since the creation of your Payrails environment. This way, you can start integrating Payrails even before having a contract with an external provider loaded into our system. Once you create your [provider integrations](provider-accounts), you should have already tested many possible flows. Hence, the only thing left is to adapt to any particular flow for that provider.

## How to use the Test PSP

You should first configure the Test PSP to be the default routing option on your workflow, or choose a specific [Meta fields](doc:meta-fields) to route your payment to it.

<img
  className="mx-auto block rounded-lg object-cover"
  src="/images/docs/orchestration/payment-acceptance/test-psp-routing-configuration.png"
  width="80%"
/>

By default, the Test PSP will always reply to your payment requests with `Success`. If you want to simulate a specific response flow, add a [Meta fields](doc:meta-fields) specifying it with one of the values of our [operation results](doc:operation-results).

<img
  className="mx-auto block rounded-lg object-cover"
  src="/images/docs/orchestration/payment-acceptance/test-psp-meta-operation-result.png"
  width="80%"
/>
