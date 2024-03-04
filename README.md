# Eulerian Marketing Platform for Google Tag Manager (GTM) S2S

Google Tag Manager connector for Eulerian Marketing Platform

## Documentation

You need to have :
- Eulerian Account - head on to [Eulerian.IO](https://www.eulerian.io) for a free account
- Create a website
- Get the generic third-party tracking collection domain, for example: https://io1.eulerian.net, the targetHost will be io1
- Implement the S2S connector in GTM-SS
- You are live ! 

### Folder structure

The source for the GTM template is in the `./template.tpl` file.

### How to deploy

1. download the file template.tpl
2. go to you google tagmanager instance
3. go to the "Templates" section
4. create new template, on the upper right side you click on the three dots & select import
5. the template is imported you just need to configure with the proper subdomain from the third-party tracking url
6. set the consent mode configuration, one of the three options :
   
   6.1 Consent is handled before us being called -> set the enoepm=1 parameter
   
   6.2 Consent through pmcat by providing the list of consented categories for the current call
   
   6.3 Consent through TCF, in this case the TCString needs to be provided through a custom variable.
   
8. save & publish the template

### Which event are mapped

#### global mapping

The following global parameters are always provided to each call sent to us as long as they exists in the original datalayer :
  o page_location mapped to url
  o page_referrer mapped to rf
  o screen_resolution mapped to ss
  o ip_override mapped to ereplay-ip
  o user_agent mapped to ereplay-ua
  o client_id mapped to euidl
  o currency mapped to currency
  o user_id mapped to uid
  o user_data.sha256_email_address mapped to email

For each call we auto-copy all additionnal parameters of the event prefixed by **ga-**

#### page_view

Standard call done

#### purchase

A transaction is registered in this case :
  o transaction_id mapped to ref
  o value mapped to amount
  o items mapped to product array & product params are prefixed by **ga-**
  
#### add_to_cart

Products listed in the items array are added to the current cart.

#### remove_from_cart

Products listed in the items array are removed from the current cart.

#### generate_lead

A lead is registered in this case :
  o transaction_id mapped to ref, if not available a random ref is created
  o value mapped to amount
  o items mapped to product array & product params are prefixed by **ga-**

#### custom events

Custom events not listed above are directly sent to actions/goals for further processing in the platform.

### WARNING

By setting up a data-collection in server-side mode like this one, you'll loose all access to the initial web-browser.
This means that some functionnalities of the Eulerian Marketing Platform won't be available because of the server-side integration, for example and not limited by :
   o Heatmap
   o Client-Side TMS
   o Identity sync for IdGraph, CookieSync, etc...
   o Real-Time access to current status of the user through javascript client-side API
   o Privacy Sandbox integration
   o and probably other use cases

## License

Apache 2.0
