## Domain/DNS Management
### DRUD DNS Records

Clients hosting in DRUD infrastructure require the following DNS records:

Name/Host/Alias | Time to Live (TTL) | Record Type | Value/Answer/Destination
------------ | ------------- | ----------- | ------------------------
Blank or @ | 3600 | A | 54.149.1.10
www | 3600 | CNAME | hosting.newmediadenver.com

Any client that hosts their site within our infrastructure must have the appropriate DNS records pointed at our proxy layer in AWS. In laymen's terms, they need to point their sites (i.e. clientsite.com and www.clientsite.com) at our infrastructure. That gives us the ability to manage which server(s) the domain points to without involving the client, which is highly beneficial for a variety of reasons (efficiency, removing the client as a dependency, etc).

Adding an Entry to Proxy Layer

* Edit the proxy: Editing the Proxy Layer Directly
* If you added the live domain to the proxy, also add it to the client's databag:
	* Under production, "url" should be set to: http(s)://www.example.com
	* Also under production, add a server alias of: - www.example.com

...
  server_aliases:
  - www.example.com
...
  url: http://www.example.com```

* If you need to add SSL certificates: Adding an SSL Certificate to a Site
Run the update jobs for the site on Jenkins. This will update the proxy layer for a single site.

### Verifying an Entry at Proxy Layer
To verify the proxy configuration is correct prior to pointing the client's domain, Edit your system /etc/hosts file with a an entry pointing the domain to one of our proxy servers:

`##`
`# Host Datbase`
`#`
`# localhost is used to configure the loopback interface`
`# when the sustem is booting. Do not change this entry`
`##`
`127.0.0.1       localhost`
`255.255.255.255 broadcasthost`
`::1             localhost`
`#proxy02.newmediadenver.com`
`54.149.1.10 example.com www.example.com`
