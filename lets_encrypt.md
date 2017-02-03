## Adding a free SSL Let's Encrypt Certificate to a Site
Let's Encrypt is a free service that allows us to generate our own secure SSL certificates. Clients can still buy their own certificate and it can still be implemented using the existing documentation, or we can provide them with a certificate that costs us nothing.

Start by running this Jenkins job: [ops-verify-dns](https://leroy.nmdev.us/job/ops-verify-dns)

If the output looks something like this:

```
Now checking the CNAME and A record for correct configuation...

There is a A record for careincommon.com. It is pointing to 107.180.48.109
 
This DNS record IS NOT CONFIGURED correctly
 
Here are the current A and CNAME values for careincommon.com:
'careincommon.com' did not have any 'CNAME' records associated with it
A records for careincommon.com:
    107.180.48.109
CNAME records for careincommon.com:
There is a CNAME record for www.careincommon.com. It is pointing to careincommon.com.
 
This DNS record IS NOT CONFIGURED correctly
 
Here are the current A and CNAME values for www.careincommon.com:
A records for www.careincommon.com:
    107.180.48.109
CNAME records for www.careincommon.com:
    careincommon.com.
Build step 'Execute shell' marked build as failure
```

where the job is claiming that the DNS records are not configured correctly, then you are either working with a site that has actually been configured improperly or you are working with a site that has not yet been pointed at our domain.

If the DNS records have been improperly configured and the site is already pointing at us, make sure to go check out this article before continuing: [Domain/DNS Management]()

If however you are launching this site for the first time and you have yet to repoint the DNS records to our stack, then we are going to setup an upstream to forward traffic to the client's existing server.

### Adding their server as an upstream
If a client already has an existing site, then we want to forward traffic from our server to theirs until we are ready to take their site live.

We would then want to:

1. Create a method on our server to forward entries to their old site
2. Check that the redirect works by using /etc/hosts
3. Point the DNS records at our server, then 
4. Repoint from the old site to the new one

Let's say we're launching careincommon.com. Open the proxy layer (databags/nmdproxy/upstream) and right under production, add a new application entry. The server IP address comes from the dig/jenkins job that you ran earlier.

```
production:
  cic:
    apps:
      www.careincommon.com: {}
    servers:
    - server 107.180.48.109:80;
```

This will allow us to accept traffic from people visiting http://www.careincommon.com and http://careincommon.com.
If the client has an existing SSL certificate, then set 'ssl_passthrough' to true and leave ssl and ssl_force undefined.
production:

```
  cic:
    apps:
      www.careincommon.com:
        ssl_passthrough: true
    servers:
    - server 107.180.48.109:80;
```

After finishing up in the proxy layer, we then open up the client secret and change the URL and server_aliases values to match this URL.

```
production:
  server_aliases:
  - www.careincommon.com
  url: https://www.careincommon.com
```
  
Now that all of our URLs match, we can run this job: [opd-deploy-by-url](https://leroy.nmdev.us/job/ops-deploy-by-url/) with Environment = production and  Client = the client's project name (e.g. cic for careincommon.com).

Go back to [Domain/DNS Management]() and use the "Verifying the entry at the proxy layer" step to setup /etc/hosts to have the domain point to us:

`54.149.1.10 www.careincommon.com careincommon.com`

If you visit the site URL right now, you should still see the old site.

If a 403 message is showing, you've done something wrong. Check the output of the proxy job you ran and see if there are any hints there.

DNS record changes are semi-unpredictable in their propagation patterns. We have a hard time telling a client exactly when their site will go live. By using this redirect strategy and creating their existing server as an upstream, we not only have the ability to apply for a certificate when we need to, but we also have direct control of when their site goes live.

Once the DNS repointing is complete (confirmed by running [ops-verify-dns](https://leroy.nmdev.us/job/ops-verify-dns)), you are done setting up your 'catcher's mitt'.

The client MUST have their DNS pointed at us before starting the next section. If it is not pointed at us, the Let's Encrypt certificate validation request will fail.

### Getting the Certificate
If the client is launching their site to a brand new URL and there is no current site, then this is the section for you!
Make sure the client has pointed their domain at our stack.

If you had to setup a redirect/new upstream above, make sure to delete the entire application block you created in the previous section.

Create the production proxy entry. Set ssl_force = True, and leave ssl undefined.

```
production:
  webcluster01:
    apps:
      www.careincommon.com:
        ssl_force: true
```
Edit the site's secret (databags/nmdhosting/SITENAME) and ensure that the URL is https and the domain name is listed in server_aliases
```
production:
  server_aliases:
  - www.careincommon.com
  url: https://www.careincommon.com
```

Then run update on the production-SITENAME jenkins job. (e.g. https://leroy.nmdev.us/job/production-cic)
The certificate willl automatically be applied for and added to the acmetool certificate renewal scheduler. And you will never have to worry about it ever again.
