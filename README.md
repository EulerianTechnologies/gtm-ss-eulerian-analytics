# Eulerian Marketing Platform for Google Tag Manager (GTM) S2S

Google Tag Manager connector for Eulerian Marketing Platform

If you don't already have an account, you can try our freemium platform through [Eulerian.IO](https://www.eulerian.io) to create a free account & declare your website.

### Documentation

1. go to you google tagmanager instance
2. go to the "Templates" section
3. look for the Eulerian Marketing Platform - GTM SS template in the gallery
4. create new template
5. the template is imported you just need to configure with the proper subdomain from the third-party tracking url

   5.1 example : if you tracking domain is : `https://et1.eulerian.net` the **targetHost** is **et1.eulerian.net**
   
   5.2 example : if you tracking domain is : `https://io1.eulerian.net` the **targetHost** is **io1.eulerian.net**

   5.3 example : if you tracking domain is : `https://sdf475.eulerian.io` the **targetHost** is **sdf475.eulerian.io**
   
6. set the consent mode configuration, one of the three options :
   
   6.1 Consent is handled before us being called -> set the enoepm=1 parameter
   
   6.2 Consent through pmcat by providing the list of consented categories for the current call
   
   6.3 Consent through TCF, in this case the TCString needs to be provided through a custom variable.
   
7. save the template
8. trigger the template with client = GA4 and on all events + custom events purchase / generate_lead.
9. publish the modifications -> you are now **live** !

### Which events are mapped

#### global mapping

The following global parameters are always provided to each call sent to us as long as they exists in the original datalayer :
- **page_location** mapped to **url**
- **page_referrer** mapped to **rf**
- **screen_resolution** mapped to **ss**
- **ip_override** mapped to **ereplay-ip**
- **user_agent** mapped to **ereplay-ua**
- **client_id** mapped to **euidl**
- **currency** mapped to **currency**
- **user_id** mapped to **uid**
- **user_data.sha256_email_address** or **user_data.em** mapped to **email**

For each call we auto-copy all additionnal parameters of the event prefixed by **ga-**

#### page_view

Standard call done

#### purchase

A transaction is registered in this case :
- **transaction_id** mapped to **ref**
- **value** mapped to **amount**
- **items** mapped to product array & product params are prefixed by **ga-**
  
#### add_to_cart

Products listed in the **items** array are added to the current cart.

#### remove_from_cart

Products listed in the **items** array are removed from the current cart.

#### view_item

Product page is viewed and the first result of the items array is sent to Eulerian.

#### generate_lead

A lead is registered in this case :
- **transaction_id** mapped to **ref**, if not available a random ref is created
- **value** mapped to **amount**
- **items** mapped to product array & product params are prefixed by **ga-**

#### custom events

Custom events not listed above are directly sent to actions/goals for further processing in the platform.

### MULTI-EVENTS

As GTM can trigger multiple events and hence our tag it can result in multiple calls on our end that can inflate the stats.

To avoid this make sure :
- you only trigger the Eulerian GTM SS only for the event you want to track
- avoid multi-event call on a single page : page_view -> view_item -> user_engagement for example only trigger on view_item if exists for example.


## Consent Management

The template supports three consent strategies, evaluated in the following priority order:

```
1. Traffic always consented (enoepm=1)     → highest priority, short-circuits all below
2. TCF v2 TCString                         → cookie source or variable source
                                               ↓ if no valid TCString found
                                             Synthetic TCString generated from GTM consent signals
3. pmcat category-based consent            → uses the category management format
```

---

### Option 1 — All traffic consented (`enoepm=1`)

Check the **"Traffic is always consented (enoepm=1)"** checkbox.

This appends `enoepm=1` to every Eulerian call, signalling that consent has been fully handled upstream before this tag fires. When this option is enabled, all other consent settings (pmcat, TCF) are ignored.

> ⚠️ **Warning:** Only use this option when you can guarantee that consent has been collected and validated **before** this tag is triggered. Misuse may result in compliance violations.

---

### Option 2 — TCF v2 Consent String

Enable the **"Enable TCF v2 consent string forwarding"** checkbox. When active, every Eulerian call will carry `gdpr=1` and `gdpr_consent=<TCString>`.

Two source modes are available:

#### 2.1 — Cookie source (`euconsent-v2`)

The template reads the TCString directly from the **`euconsent-v2`** cookie set by the CMP on the user's browser.

> **Prerequisite:** Your GTM Server-Side container must be served from a **first-party subdomain** (e.g. `gtm.yourdomain.com`). On a third-party domain (e.g. `googletagmanager.com`) the browser will not forward first-party cookies and this mode will silently fail.

An optional **Fallback TCString** field is available. If the cookie is absent or contains an invalid value, the template will attempt to use the fallback string you provide before resorting to synthetic generation (see below).

#### 2.2 — Variable source (GTM SS variable)

The template reads the TCString from any **GTM Server-Side variable** you map in the tag configuration. Use the **"GTM Variable holding the TCString"** field to select or reference your variable.

This is the recommended approach when:
- The TCString is pushed to the dataLayer client-side and forwarded as an event parameter (e.g. via a `tcString` event parameter mapped to an Event Data variable)
- A custom GTMSS client extracts it from a request header or query parameter
- You have a custom JavaScript variable that resolves the TCString

> The variable is resolved at tag execution time by the GTM SS sandbox. Any GTM SS variable type is supported.

---

### TCString Fallback — Synthetic Generation

If neither the cookie nor the variable source provides a valid TCString (absent, empty, or too short), the template **automatically generates a synthetic TCString** derived from GTM's native consent signals via `isConsentGranted()`.

The synthetic string is a minimal valid **TCF v2 Core String** with the following characteristics:

- Purposes are mapped from GTM consent types as follows:

| TCF Purpose | GTM Consent Signal |
|---|---|
| Purpose 1 | `ad_storage` OR `analytics_storage` OR `functionality_storage` |
| Purpose 2 | `ad_storage` |
| Purpose 3 | `ad_storage` AND `ad_personalization` |
| Purpose 4 | `ad_storage` AND `ad_personalization` |
| Purpose 5 | `personalization_storage` |
| Purpose 6 | `personalization_storage` |
| Purposes 7–10 | `analytics_storage` |
| Purposes 11–24 | `false` (no GTM mapping) |

- **All vendors** (IDs 1–1700) are granted consent via range encoding
- **No Legitimate Interest** — LI sections are zeroed
- `CmpId = 0` — this is a synthetic string, not issued by a registered CMP

> ⚠️ **Important:** The synthetic TCString is a best-effort fallback for sites without a CMP or where the TCString is temporarily unavailable. It is **not a substitute for a real CMP** on sites legally required to collect explicit consent under GDPR. It should be treated as a signal-forwarding mechanism only.

---

### Option 3 — Consent via pmcat categories

Fill in the **"List of pmcat values"** field with the IDs of consented categories for the current call (e.g. `1-3`).

This option is only available when `enoepm=1` and TCF is not enabled is not checked.

---

### Consent Decision Flow

```
Tag fires
    │
    ▼
enoepm=1 checked?
  YES → send enoepm=1, done ✅
  NO  ↓
      │
      ├─ tcfEnabled checked?
      │    YES ↓
      │        ├─ source = cookie
      │        │     Read euconsent-v2 cookie
      │        │     Valid? → use it ✅
      │        │     No?   → check fallback TCString
      │        │               Valid? → use it ✅
      │        │               No?   → generate synthetic ✅
      │        │
      │        └─ source = variable
      │              Resolve GTM SS variable
      │              Valid? → use it ✅
      │              No?   → generate synthetic ✅
      │
      └─ tcfEnabled NOT checked
           pmcat set? → send pmcat ✅
           pmcat not set? → call sent without consent signal
```
> ⚠️ **Important:** - make sure the CMP is blocking navigation to avoid having a waiting consent status idling, you **MUST** provide the consent on all calls, if you set-up TCF and the CMP is non-blocking then traffic will be considered consented by default, you can use the Variable or defaultTCString set-up to change this behaviour.
> We recommend in any case to have a blocking CMP.

### WARNING

By setting up a data-collection in server-side mode like this one, you'll loose all access to the initial web-browser.
This means that some functionnalities of the Eulerian Marketing Platform won't be available because of the server-side integration, for example and not limited by :
   - Heatmap
   - Post-impression tracking
   - Client-Side TMS
   - Identity sync for IdGraph, CookieSync, etc...
   - Real-Time access to current status of the user through javascript client-side API
   - Privacy Sandbox integration
   - Client-Hints management
   - and probably other use cases

So make sure you know what you are doing and know what you want to achieve.

### SUPPORT

We provide limited support through our free offering, if your setup is more complex we can provide consulting expertise and/or work with your expert agencies of your choice.
This means that the current plugin works for most cases but not all cases, so please understand your own setup and make sure it makes sense regarding what we are collecting so that you can take the most of our platform. Enjoy !

## License

Apache 2.0
