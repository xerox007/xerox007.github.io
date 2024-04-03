---
title: "Configure custom domain .com.np with github pages using cloudflare dns "
categories:
  - blog
tags:
  - github
  - github-pages
  - cloudflare
  - .com.np domain
---

This is a post regarding the integration of custom domain (.com.np) for github pages using cloudflare dns.

## Prerequisites:

- You must have a valid approved `.com.np` domain from [Mercantile][np-registration]. There are a lots of articles online regarding the documents required and letter to be written.
- A cloudflare account
- A github pages site that is up and running and is accessible using the `<username>.github.io` URL. You can use this [Jekyll Theme Starter][theme-gen-link]

## Problem faced:

After the `<username>.github.io` github pages site is running, we can refer the [github article][githubpages-custom-domain] to get more info on custom domain configuration. From the article, we can get that we need to make `A` records and `CNAME` records for the github servers where the github pages is hosted.

```
A Records
----------
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

But, in the `.com.np` registration section, we can see that we cannot add IP Addresses and we must add the nameservers. As mentioned in the site, Ip addresses are strictly prohibited. So to solve this we will be using Cloudflare's DNS Server names in the `.com.np` registration section and in the cloudflare dns records, we will add those A and CNAME records.
![][comnp-img]

## Solution:

Now, we have to create the `A and CNAME` records in the cloudfare's dns interface. `A` records are used to point the root domain `<firstname><lastname>.com.np` to github pages server ip addresses while `CNAME` records are used as alias for the `www.` or any other subdomains.

![][dns-interface-cloudfare-img]

We can see that the root domain `<firstnamelastname>.com.np` is pointed to four different ip address of githubpages creating 4 different `A` records. Also, we can see a `CNAME` record with subdomain `www` being pointed to or aliased to root domain `<firstnamelastname>.com.np`.

**Don't use the proxied option while creating the A records. Use DNS only. Also, keep the TTL values below 5 min for faster DNS propagation**

The DNS propagation will take 24-72 hours. Have some patience.

This results in **cloudflare nameservers** creation. We have to use the generated nameservers from cloudflare in the `.com.np registration` interface.

In the GitHub Pages > Custom domain section, it is better to use the `www.<firstname><lastname>.com.np` subdomain instead of the root domain.
Also, before enforcing the `HTTPS`, DNS check should be successful.

First, we can use `Full` option in the SSL/TLS Section in the cloudflare interface for secure connection from browser to Cloudflare and from Cloudflare to Githubpages. Then, check the `Enforce HTTPS` option in github pages settgins.

![][githubpages-settings-img]

## Result:

![][secure-connection-img]

We used Cloudflare to create `A and CNAME` records in their DNS records which resulted in name server generation. This name server is then used in the .com.np registration interface.

When the user hits `www.<firstname><lastname>.com.np` in the browser, the cloudflare's name server handles this request to resolve the domain name into the IP addresses of Githubpages by looking into the `A` records and `CNAME` records. Finally, our webpage is served by the githubpages.

[np-registration]: https://register.com.np/
[theme-gen-link]: https://github.com/mmistakes/mm-github-pages-starter/generate
[githubpages-custom-domain]: https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-a-subdomain
[comnp-img]: /assets//images/npdomain-cloudflare-githubpages/np-interface.png
[dns-interface-cloudfare-img]: /assets//images/npdomain-cloudflare-githubpages/dns-interface-cloudfare.png
[githubpages-settings-img]: /assets//images/npdomain-cloudflare-githubpages/githubpages-settings-custom-domain.png
[secure-connection-img]: /assets//images/npdomain-cloudflare-githubpages/secure-connection.png
