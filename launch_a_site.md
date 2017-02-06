# Launch a Website

The order of operations is to:

1. Create a method on our server to forward entries to their old site (if applicable)
2. Check that the redirect works by using /etc/hosts
3. Point the DNS records at our server
4. Repoint from the old site to the new one on our infrastructure

**DNS repointing should be done as soon as possible. If you wait until the end of the project, the launch process is much less reliable/predictable. Complete all the steps through to #Point-the-DNS-at-DRUD the second the client has paid at the beginning of the project.**

## Launching a Pre-Existing Site / Creating an Upstream
Think about creating an upstream proxy entry like setting up a catcher's mitt. We catch/receive inbound traffic and then toss/forward that traffic to the pre-existing site's old server. The site should look the same before starting this and after completing it.

DNS record changes are semi-unpredictable in their propagation patterns. We have a hard time telling a client exactly when their site will go live. By using this redirect strategy and creating their existing server as an upstream, we not only have the ability to apply for a certificate when we need to, but we also have direct control of when their site goes live.


### Forwarding Traffic to an Existing Server

Let's say we're launching careincommon.com. We would open the proxy layer (databags/nmdproxy/upstream) and right under production, add a new application entry. The server IP address comes from the dig/jenkins job that you ran earlier.

```
production:
  cic:
    apps:
      www.careincommon.com: {}
    servers:
    - server 107.180.48.109:80;
```

This will allow us to accept traffic from people visiting http://www.careincommon.com and http://careincommon.com.
If the client has an existing SSL certificate, then set 'ssl\_passthrough' to true and leave 'ssl' and 'ssl\_force' undefined.

```
production:
  cic:
    apps:
      www.careincommon.com:
        ssl_passthrough: true
    servers:
    - server 107.180.48.109:80;
```
You are now done setting up your traffic forwarding catcher's mitt. You may move onto the /etc/hosts verification step.

## Enable the Upstream
Now that all of our URLs match, we can run this job: [ops-deploy-by-url](https://leroy.nmdev.us/job/ops-deploy-by-url/) with `Environment = production` and `Client = the client's project name (e.g. cic for careincommon.com)`.

Verify that your changes worked using /etc/hosts by following the steps in [Verifying an Entry at Proxy Layer](proxy_cheatsheet.md#Verifying-an-Entry-at-Proxy-Layer)

## Verifying a Site Using /etc/hosts
Setup /etc/hosts to force the domain point to us:

`54.149.1.10 www.careincommon.com careincommon.com`

If you visit the site URL right now, you should still see the old site.

If a 403 message is showing, you've done something wrong. Check the output of the proxy job you ran and see if there are any hints there.

If your verification steps check out, :thumbsup:, move on. If not, double-check that:

- The jenkins job ran successfully
- The databags/nmdhosting/SITENAME production `server_alias` and `url` values match the proxy entry in databags/nmdproxy/upstream

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

If you are launching a Drupal site, you can go ahead and save the vault secret (by quitting out of your editor) and skip to #Apply-the-Changes.

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

## SSL
Once the DNS repointing is complete (confirmed by running [ops-verify-dns](https://leroy.nmdev.us/job/ops-verify-dns)), and the client site is loading successfully, you are done setting up your 'catcher's mitt'.

The client MUST have their DNS pointed at us before starting the next section. If it is not pointed at us, the Let's Encrypt certificate validation request will fail as will the drudfab production-{site} deploy.

