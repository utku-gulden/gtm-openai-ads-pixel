# OpenAI Ads Pixel: Google Tag Manager Community Template

A Google Tag Manager Community Template for the OpenAI Ads Measurement Pixel (`oaiq`). It lets you install, initialize and send conversion events from your website without writing custom HTML, modelled on the UX of the Meta Pixel and TikTok Pixel templates.

> **Source of truth**
> - Pixel docs: https://developers.openai.com/ads/measurement-pixel
> - Supported events: https://developers.openai.com/ads/supported-events

> **Created by:** [Utku Gülden](https://www.linkedin.com/in/utku-gulden/)
> · Brand id: `utkugulden`
> · Community-maintained, not affiliated with OpenAI or Google.

---

## What it does

* Loads the OpenAI pixel SDK from `https://bzrcdn.openai.com/sdk/oaiq.min.js`.
* Initializes the pixel with your Pixel ID, idempotent across multiple tag fires.
* Sends one of ten standard events (`page_viewed`, `contents_viewed`, `items_added`, `checkout_started`, `order_created`, `lead_created`, `registration_completed`, `appointment_scheduled`, `subscription_created`, `trial_started`) or a custom event with your own name.
* Maps fields to the four official OpenAI data shapes: `contents`, `customer_action`, `plan_enrollment`, `custom`.
* Supports a single product, a multi-product `contents[]` array sourced from a GTM variable, or no contents at all.
* Forwards `event_id` and `custom_event_name` inside `eventOptions`.
* Allows arbitrary extra fields via Additional Properties, merged into `eventProps` after standard fields.

## Installation

### Option A: Single container import

1. In your GTM workspace, open **Templates** then **New** under Tag Templates.
2. Click the three-dot menu (top right) and choose **Import**.
3. Select `template.tpl` from this repository.
4. Save the template.
5. Create a new tag using the imported template.

### Option B: Community Template Gallery

Available after this template is approved by Google. Once published, search for "OpenAI Ads Pixel" in the Gallery from any container.

---

## Setup walkthrough

### 1. Get your Pixel ID

Go to [ads.openai.com](https://ads.openai.com), open the **Conversions** tab and copy the Pixel ID.

### 2. Create the page-load tag

* **Tag Type:** OpenAI Ads Pixel
* **Pixel ID:** paste your value (or reference a constant variable)
* **Event Name:** Standard, default `page_viewed`
* **Trigger:** All Pages

This tag fires on every page, lazily loads the SDK, initializes the pixel and sends `page_viewed`.

### 3. Create event tags as needed

For each conversion event, create a new tag with the matching trigger and Event Name.

| Trigger                       | Event Name (Standard)   |
| ----------------------------- | ----------------------- |
| Add to Cart click             | `items_added`           |
| Checkout start                | `checkout_started`      |
| Order confirmation page       | `order_created`         |
| Lead form submission          | `lead_created`          |
| Signup complete               | `registration_completed`|
| Subscription successful       | `subscription_created`  |

### 4. Money values

`Amount` and per-item `amount` are integers, per the [OpenAI docs](https://developers.openai.com/ads/measurement-pixel) ("Use integer values for `amount` and `quantity`"). Decimal inputs are rounded to the nearest whole number: `23.49` becomes `23`, `23.50` becomes `24`.

`Currency` is the three letter ISO 4217 code (`USD`, `EUR`, `GBP`, `TRY`, `JPY`, ...). Required whenever `Amount` is set.

### 5. Multi-product orders

For an order with more than one product, switch the Contents mode to **Multiple Contents** and point the field at a GTM Custom JavaScript Variable that returns the array.

Example variable that converts a GA4 `dataLayer` items array into the OpenAI shape:

```js
function() {
  var items = {{DLV - ecommerce.items}};
  if (!items) return [];
  return items.map(function(it) {
    return {
      id: it.item_id,
      name: it.item_name,
      content_type: 'product',
      quantity: it.quantity,
      amount: Math.round(it.price),
      currency: it.currency || 'USD'
    };
  });
}
```

The template coerces each row to the OpenAI schema: string fields stay strings, `quantity` and `amount` become integers, `currency` is uppercased. Items that produce no fields are dropped.

---

## Standard events reference

| Event name                | Data shape         |
| ------------------------- | ------------------ |
| `page_viewed`             | `contents`         |
| `contents_viewed`         | `contents`         |
| `items_added`             | `contents`         |
| `checkout_started`        | `contents`         |
| `order_created`           | `contents`         |
| `lead_created`            | `customer_action`  |
| `registration_completed`  | `customer_action`  |
| `appointment_scheduled`   | `customer_action`  |
| `subscription_created`    | `plan_enrollment`  |
| `trial_started`           | `plan_enrollment`  |

For custom events, you pick the data shape yourself from the **Data shape** dropdown.

---

## Custom events

Custom event names follow strict rules:

* Lowercase letters, numbers, underscore and hyphen only: `^[a-z0-9_-]{1,64}$`.
* Cannot reuse any standard event name (e.g. you cannot create a custom event called `order_created`).
* 1 to 64 characters.

For custom events, pick the closest data shape from the dropdown:

* `contents`: events with product or item lists.
* `customer_action`: events with just amount and currency, or no money values.
* `plan_enrollment`: subscription-style events with a `plan_id`.
* `custom` (default): anything else.

The custom event name is sent both as the measure event name and inside `eventOptions.custom_event_name`, matching the pattern shown in the official docs.

---

## Additional Properties

Use the **Additional Properties** key-value table to send arbitrary extra fields. Each non-empty row adds one entry to `eventProps`. Standard fields (`amount`, `currency`, `plan_id`, `contents`, `type`) take priority and are not overwritten by an Additional Property with the same key.

---

## Permissions

The template declares three Web permissions:

| Permission        | Why                                                                  |
| ----------------- | -------------------------------------------------------------------- |
| `logging`         | Console logs in the GTM debug environment.                           |
| `inject_script`   | Loads `https://bzrcdn.openai.com/sdk/oaiq.min.js`.                   |
| `access_globals`  | Reads, writes and executes the `oaiq` and `oaiq.q` globals.          |

When importing the template, GTM asks you to approve these. They cover everything the template needs and nothing more.

---

## Repository layout

```
.
├── LICENSE                    Apache 2.0
├── README.md                  this file
├── examples/
│   └── example-config.json    reference fixture used in tests
├── metadata.yaml              Gallery submission manifest
├── screenshots/
│   └── icon.png               96x96 brand thumbnail
└── template.tpl               the GTM template file
```

---

## Contributing

Issues and pull requests are welcome. For non-trivial changes please open an issue first.

## License

Apache 2.0. See [LICENSE](LICENSE).
