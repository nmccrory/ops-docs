# Launch a Website

The order of operations is to:

- Create a method on our server to forward entries to their old site (if applicable)
- Check that the redirect works by using /etc/hosts
- Point the DNS records at our server
- Repoint from the old site to the new one on our infrastructure

**DNS repointing should be done as soon as possible. If you wait until the end of the project, the launch process is much less reliable/predictable. Complete all the steps through to #Point-the-DNS-at-DRUD the second the client has paid at the beginning of the project.**

## Launching a Pre-Existing Site / Creating an Upstream
Think about creating an upstream proxy entry like setting up a catcher's mitt. We catch/receive inbound traffic and then toss/forward that traffic to the pre-existing site's old server. The site should look the same before starting this and after completing it.

DNS record changes are semi-unpredictable in their propagation patterns. We have a hard time telling a client exactly when their site will go live. By using this redirect strategy and creating their existing server as an upstream, we not only have the ability to apply for a certificate when we need to, but we also have direct control of when their site goes live.


### Forwarding Traffic to an Existing Server

Let's say we're launching careincommon.com. We would open the proxy layer (databags/nmdproxy/upstream) and right under production, add a new application entry. The server IP address comes from the dig/jenkins job that you ran earlier.

```yaml
production:
  cic:
    apps:
      www.careincommon.com: {}
    servers:
    - server 107.180.48.109:80;
```

This will allow us to accept traffic from people visiting http://www.careincommon.com and http://careincommon.com.

If the client has an existing SSL certificate, then your work is a little more complicated.
You will need to obtain a copy of the existing certificate from the client, then follow the steps as though you were installing a pre-purchased SSL certificate for a standard site launch detailed in the [Pre-defined SSL Certificates](ssl.md#pre-defined-ssl-certificates) section.

```yaml
production:
  cic:
    apps:
      www.careincommon.com:
        ssl_force: true
        ssl: "careincommon.com"
    servers:
    - server 107.180.48.109:80;
```
_Note_: The `ssl_force` is theoretically optional, so if you notice issues with us enforcing the SSL redirect at our proxy layer, try removing it.

<!---
It's **possible** that we might just be able to use let's encrypt on our own server and pass the traffic on http, but I think it would end up causing a redirect loop? More testing is needed.
As for the passthrough below
If the client has an existing SSL certificate, then set 'ssl\_passthrough' to true and leave 'ssl' and 'ssl\_force' undefined.

```yaml
production:
  cic:
    apps:
      www.careincommon.com:
        ssl_passthrough: true
    servers:
    - server 107.180.48.109:80;
```
--->
You are now done setting up your traffic forwarding catcher's mitt. You shall now pass -->

## Enable the Upstream
Now that all of our URLs match, we can run this job: [ops-deploy-by-url](https://leroy.nmdev.us/job/ops-deploy-by-url/) with `Environment = production` and `Client = the client's project name (e.g. cic for careincommon.com)`.


## Verify and Troubleshoot
**Verify that your changes worked** using /etc/hosts by following the steps in [Verifying a Proxy Entry](proxy_cheatsheet.md#Verifying-a-Proxy-Entry)

After modifying /etc/hosts, when you visit the site URL, you should still see the old site.

If you run a curl command against the site's URL (or load the site in Chrome and examine the header there):

```bash
➜  ~ curl -I https://www.newmedia.com
HTTP/1.1 301 Moved Permanently
Server: nginx
Date: Tue, 07 Feb 2017 20:36:20 GMT
Content-Type: text/html
Content-Length: 178
Connection: keep-alive
Location: https://www.newmediadenver.com/
X-Proxy-Host: proxy13
```
Look for a value called `X-Proxy-Host`. If you see this value, it means that the request was routed through our proxy servers. You are looking at the site as it would appear if the DNS was pointed at us.

If a 403 message is showing, you've done something wrong. :expressionless: Check the output of the proxy job you ran and see if there are any hints there in error messages.

If your verification steps check out, :thumbsup:, move on. If not, double-check that:

- The jenkins update job ran successfully
- The `databags/nmdhosting/SITENAME` production `server_alias` and `url` values match the proxy entry app name in `databags/nmdproxy/upstream`
- If the job failed at the Let's Encrypt step, read through [Let's Encrypt](lets_encrypt.md) for a sanity check.

## Point the DNS at DRUD
#### DRUD DNS Records
Clients hosting in DRUD infrastructure require the following DNS records:

Name/Host/Alias | Time to Live (TTL) | Record Type | Value/Answer/Destination
------------ | ------------- | ----------- | ------------------------
Blank or @ | 3600 | A | 54.149.1.10
www | 3600 | CNAME | hosting.newmediadenver.com

Any client that hosts their site within our infrastructure must have the appropriate DNS records pointed at our proxy layer in AWS. In laymen's terms, they need to point their sites (i.e. clientsite.com and www.clientsite.com) at our infrastructure. That gives us the ability to manage which server(s) the domain points to without involving the client, which is highly beneficial for a variety of reasons (efficiency, removing the client as a dependency, etc).

####Mail Server Considerations
While many domain management interfaces will use MX to control mail separately, some sytems, like GoDaddy, rely on a "mail" CNAME. If the primary host is pointed elsewhere, the mail will be as well. To address this (at least within GoDaddy), remove the mail CNAME and add an A record for "mail" that will be pointed at the previous host IP. If there is a cPanel, that should also be updated to the mail host that was just created. Unless familiar with the unique system, it does not hurt to call the hosting entity and confirm settings.

####Client Communication
Here is the template for the client communication: [DNS Change Client Communication.txt](files/dns_email.txt)

## Connecting an Internal Site to a Client URL
These steps create an association between the internal application (e.g. the new site) and the client's URL.

Open up the client secret `drud secret edit databags/nmdhosting/SITENAME` and set the `url` and `server_aliases` values to match this URL. When you launch a site, you want only one server alias per site.

```yaml
production:
  server_aliases:
  - www.careincommon.com
  url: https://www.careincommon.com
```
Tips:

- Be careful not to put a trailing slash at the end of `url`.
- `server_aliases` should not have 'http(s)://' in it.

If you are launching a Drupal site, you can go ahead and save the vault secret (by quitting out of your editor) and skip to [Get It Done](launch_a_site.md#Get-It-Done).

---
**If you are launching a WP site**, your work is (sadly) not complete. Update the `search_replace` array. Add the temporary production URL to the `search_replace` key. For example:

```yaml
search_replace:
    - http://1feeprod.nmdev.us
    - https://1feeprod.nmdev.us
    - http://1fee.nmdev.us
    - https://1fee.nmdev.us
```
_Note: Internally our sites run on HTTPS so even if the live site is not using HTTPS be sure to add `search_replace` values for both HTTP and HTTPS urls in the `search_replace` key. The goal is to capture every possible internal/development URL to eliminate them from the production site._

Close and save the vault secret (e.g. save+quit out of your editor).

## Proxy Layer
Once the DNS repointing is complete (confirmed by running [ops-verify-dns](https://leroy.nmdev.us/job/ops-verify-dns)), and the client site is loading successfully using the /etc/hosts spoof trick detailed in [Verifying a Proxy Entry](proxy_cheatsheet.md#verifying-a-proxy-entry), then you are ready to update the proxy layer to stop forwarding traffic to the old/upstream server and to instead point it at our server

If the client has provided a pre-purchased SSL certificate, follow the installation steps detailed in the [Pre-defined SSL Certificates](ssl.md#pre-defined-ssl-certificates) section. Once complete, you may skip ahead to [Get it Done](#get-it-done).

If the client has not provided an SSL certificate, that's OK, we can provide them with a free one using Let's Encrypt.

**tl;dr:** Set the `ssl_force` flag to `true` in the proxy layer and make sure that `ssl` and `ssl_passthrough` are not defined.

More detailed instructions can be found on the [Let's Encrypt](ssl.md#lets-encrypt-certificates) page.

## Get It Done
If you've done everything correctly up until this point, then you are a rockstar, and you should just be able to run and update on your site (https://leroy.nmdev.us/production-SITENAME).

Afterwards, try loading the client site in an incognito window (or a different browser) and click on the server certificate details by clicking on the lock in the address bar. If you don't see a padlock or https, then the certificate didn't take. If you are updating an existing SSL certificate, just make sure the new certificate expiration date is correct in the certificate details pane.

if job == success: :beers:

if job == failure: Double-check the suggested steps in the [Verify and Troubleshoot](launch_a_site.md#Verify-and-Troubleshoot) section and if that doesn't work, start back [at the top](launch_a_site.md#).