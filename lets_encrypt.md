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
