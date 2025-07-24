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
