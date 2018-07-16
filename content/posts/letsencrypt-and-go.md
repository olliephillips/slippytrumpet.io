+++
title = "Let's Encrypt and Go"
date = "2016-10-31T18:43:50+01:00"
description = "Recently I implemented TLS using Let's Encrypt for two Go applications."
tags = [
    "golang",
    "letsencrypt",
    "TLS"
]
categories = [
    "programming",
]

+++

## Recently I implemented TLS using Let's Encrypt for two Go applications. I thought I'd outline the process I followed and mention a couple of issues I had.

[Let's Encrypt](https://letsencrypt.org/) is a fabulous service. In making TLS freely accessible to the masses it's helping make the Internet a more secure place.

For Go applications, there's no automatic install and renewal route, the process you follow to obtain a certificate takes you via the "[standalone](http://letsencrypt.readthedocs.io/en/latest/using.html#standalone)" option. 

Alternatively, you can use a helper package such as [ACMEWrapper](https://github.com/dkumor/acmewrapper) in your application; add a bit of code and ACMEWrapper automatically handles certificate creation and more importantly renewal, since Let's Encrypt certificates expire every 90 days. 

I didn't use ACMEWrapper. In fact I only learned of this package after my TLS deployments were done. But even without one of these helper packages, the process of certificate creation and renewal via the "standalone" route is straightforwards:

* Install the Let's Encrypt client "Certbot" on your server;
* Run a single command to generate the certificates for the required domain;
* Either copy, or symlink, the required key and certificate files to your application.

## Unbind all your things

To create a certificate for a domain, Let's Encrypt needs to validate you control the domain. 

If you're generating the certificates on the server on you which you want to use them, this requires that port 80 and/or 443 are available to Certbot. This means you'll have to unbind anything running on those ports, which typically means you'll have to stop your application(s) for the duration of the certificate creation process.

Certbot makes you aware if it can't use the ports it needs, so this is a minor obstacle and quickly overcome. I only mention it in case your application already has users - you'll want to consider when best to run Certbot.

No big deal but my first issue - my application was already live, and had to go offline for about 20 seconds to free those ports for Certbot.

## I think your SSL has expired?

Certbot "standalone" creates four files after successfully completing.

```
cert.pem
chain.pem
fullchain.pem
privkey.pem
```
<br/>
Your go application needs two - but which two?

You need a private key, so `privkey.pem` is an obvious pick. You also need the certificate itself, so `cert.pem` is a reasonable choice too. 

But you need to use the less obviously named `fullchain.pem` for the certificate in your Go application. This file combines server, root and intermediate certificates into one.

Choose the more obvious `cert.pem` (as I did) and though the certificate may look good to you and many others, you'll get reports from people using some browsers saying your SSL has expired or your TLS is broken, and they can't or won't use your site. 

Hard to debug.

So why `fullchain.pem`? Though my knowledge is limited, broadly it seems to relate to the fact that Let's Encrypt certificates are currently cross signed by Identrust to ensure browser acceptance. Let's Encrypt is not as yet a root authority so needs this, especially to appear trusted in older browsers.

















