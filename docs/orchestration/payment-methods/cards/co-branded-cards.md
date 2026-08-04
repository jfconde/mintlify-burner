# Co-branded cards
Some cards carry two payment networks at once — for example a **mada + Mastercard** card in Saudi Arabia or a **Cartes Bancaires + Visa** card in France. For these _co-branded_ (also called _co-badged_) cards, the cardholder is entitled to choose which network processes the payment, and each brand must be presented with equal visual weight.

When you accept cards through the [Payrails SDKs](/docs/orchestration/checkout-sdks/index), this journey is handled for you. The SDK detects co-branded cards from the BIN as the shopper types, shows both brand logos in the card-number field, and lets the shopper pick a brand. The shopper's chosen network is submitted to Payrails as the `preferredScheme` so the payment is routed accordingly.

## How it works

1. As the shopper enters their card number, the SDK performs a BIN lookup and resolves the available networks.
2. If the card is single-branded, only one brand applies and no selector is shown.
3. If the card is co-branded, the SDK renders **both** scheme logos inside the card-number field and exposes the available schemes (`cardSchemes`) so a brand selector can be shown.
4. The display order and the pre-selected brand follow the **merchant's** `preferredSchemes`, which is initially registered based on your merchant preference; if none is configured, the card's local network is preferred.
5. The **shopper's** choice is sent to Payrails as `paymentInstrumentData.preferredScheme` (a single network code such as `cartesbancaires` or `mada`).

<Note>
  ### `preferredSchemes` (merchant) vs `preferredScheme` (shopper)

  These two names look alike but refer to different things:

  - `preferredSchemes` (plural) is the **merchant's** preference — an ordered list registered per account that drives the brand display order and which brand is pre-selected. You don't set it in code; it's delivered to the SDK on initialization.
  - `preferredScheme` (singular) is the **shopper's** choice — the single brand the shopper picks in the card brand selector, submitted to Payrails on the payment payload as `paymentInstrumentData.preferredScheme`.

</Note>

<Note>
  ### Availability

  The co-branded card journey is available in the [**web SDK** from version `5.48.0`](https://www.npmjs.com/package/@payrails/web-sdk/v/5.48.0) and the [**React Native SDK** from version `2.10.0`](https://www.npmjs.com/package/@payrails/react-native-sdk).
</Note>

<Note>
  ### Enabling co-branded cards

  The co-branded journey is enabled per account and runs on the SDK's secure-fields rendering path. Your scheme preference order (`preferredSchemes`) is configured on the Payrails side and delivered to the SDK automatically — you don't pass it in code. Reach out to your Payrails contact to enable it for your account.
</Note>

## By integration type

The journey behaves the same across Payrails SDKs; the code examples below are for the web SDK. How much you need to build depends on which integration you use:

| Integration           | Brand selector      | What you do                                                                                                                          |
| :-------------------- | :------------------ | :----------------------------------------------------------------------------------------------------------------------------------- |
| **Drop-in**           | Rendered by the SDK | Nothing — the selector appears automatically and `preferredScheme` is submitted for you.                                             |
| **Card Form element** | Rendered by the SDK | Nothing required. Optionally read `cardSchemes` from `onChange` to react to the selection.                                           |
| **Secure Fields**     | You render it       | Read `cardSchemes` from the card-number `CHANGE` event, render your own selector, and add `preferredScheme` to your payment payload. |

### `CardScheme` shape

Both the Card Form `onChange` event and the Secure Fields `CHANGE` event expose the available networks as a `cardSchemes` array. It is only present for co-branded cards.

```ts
interface CardScheme {
  code: string;       // network code, e.g. "visa" — submit this as preferredScheme
  name: string;       // display label, e.g. "Visa", "mada"
  logoUrl?: string;   // scheme logo asset
  selected: boolean;  // the pre-selected default (exactly one is true)
}
```

## Drop-in and Card Form

With Drop-in and the Card Form element, the SDK renders a "Card Brand" selector beneath the form whenever a co-branded card is detected, and submits the shopper's choice automatically. No extra integration code is required.

You can tailor the selector's wording and look to match your checkout:

```js
const cardForm = payrails.cardForm({
  showCardHolderName: true,
  translations: {
    labels: {
      cardBrandSelectorTitle: 'Choose your card brand',
      cardBrandSelectorSubtitle: 'You can choose which network to pay with. This is optional.',
    },
  },
  styles: {
    // All tokens are optional and applied uniformly to every brand tile
    // (equal visual treatment is a regulatory requirement).
    cardBrandSelector: {
      labelColor: '#1e293b',
      subtitleColor: '#64748b',
      tileBackground: '#ffffff',
      tileBorderColor: '#e2e8f0',
      tileBorderRadius: '8px',
      tileTextColor: '#1e293b',
      selectedBorderColor: '#2563eb',
      selectedCheckColor: '#2563eb',
      focusRingColor: '#2563eb',
    },
  },
  events: {
    // Fires whenever the shopper switches brand.
    onPreferredSchemeChanged: ({ preferredScheme, cardSchemes }) => {
      console.log('Shopper chose', preferredScheme, cardSchemes);
    },
    // The co-branded schemes also ride the standard change event.
    onChange: (e) => {
      if (e.cardSchemes) {
        console.log('Co-branded card detected:', e.cardSchemes);
      }
    },
  },
});
```

The same `translations` and `styles.cardBrandSelector` options apply to the cards section of a Drop-in configuration.

## Secure Fields

With Secure Fields you build your own form and pay button, so you also render your own brand selector. The SDK still detects the co-branded card and shows the dual logos in the card-number field (when `enableCardIcon` is on); your job is to read the schemes, track the shopper's choice, and pass it on the payment payload.

```js
const container = payrails.collectContainer({ containerType: 'COLLECT' });

const cardNumber = container.createCollectElement({
  type: ElementType.CARD_NUMBER,
  enableCardIcon: true, // shows both brand logos inside the field
});

let selectedScheme = null;

cardNumber.on('CHANGE', (e) => {
  if (e.cardSchemes) {
    // Co-branded card: render your own selector from e.cardSchemes
    renderBrandSelector(e.cardSchemes, (code) => { selectedScheme = code; });
    selectedScheme = e.cardSchemes.find((s) => s.selected)?.code ?? null;
  } else {
    // Single-branded (or BIN not yet resolved): hide your selector
    hideBrandSelector();
    selectedScheme = null;
  }
});

cardNumber.mount('#card-number');

// On submit, collect the card data and attach the chosen scheme.
async function pay() {
  const collected = await container.collect();
  const paymentInstrumentData = selectedScheme
    ? { ...collected, preferredScheme: selectedScheme }
    : collected;

  // Send paymentInstrumentData on your authorize request to Payrails.
}
```
