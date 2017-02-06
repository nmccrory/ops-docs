# Launch a Website

The order of operations is to:

1. Create a method on our server to forward entries to their old site (if applicable)
2. Check that the redirect works by using /etc/hosts
3. Point the DNS records at our server
4. Repoint from the old site to the new one on our infrastructure


## Launching a Pre-Existing Site / Creating an Upstream
Think about an upstream like a catcher's mitt. We catch/receive inbound traffic and then toss/forward the traffic on to the existing site's old server. The site should look the same before starting this and after completing it.

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

After finishing up in the proxy layer, we then open up the client secret and change the URL and server_aliases values to match this URL.

```
production:
  server_aliases:
  - www.careincommon.com
  url: https://www.careincommon.com
```
  
Now that all of our URLs match, we can run this job: [ops-deploy-by-url](https://leroy.nmdev.us/job/ops-deploy-by-url/) with `Environment = production` and `Client = the client's project name (e.g. cic for careincommon.com)`.

### Verifying a Site Using /etc/hosts
Go back to [Domain/DNS Management]() and use the "Verifying the entry at the proxy layer" step to setup /etc/hosts to force the domain point to us:

`54.149.1.10 www.careincommon.com careincommon.com`

If you visit the site URL right now, you should still see the old site.

If a 403 message is showing, you've done something wrong. Check the output of the proxy job you ran and see if there are any hints there.

### Point the DNS at DRUD

Once the DNS repointing is complete (confirmed by running [ops-verify-dns](https://leroy.nmdev.us/job/ops-verify-dns)), you are done setting up your 'catcher's mitt'.

The client MUST have their DNS pointed at us before starting the next section. If it is not pointed at us, the Let's Encrypt certificate validation request will fail as will the drudfab production-{site} deploy.

