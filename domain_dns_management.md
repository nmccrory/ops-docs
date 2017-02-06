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
	
```
production:
...
  server_aliases:
  - www.example.com
...
  url: http://www.example.com
```

* If you need to add SSL certificates: Adding an SSL Certificate to a Site
Run the update jobs for the site on Jenkins. This will update the proxy layer for a single site.

### Verifying an Entry at Proxy Layer
To verify the proxy configuration is correct prior to pointing the client's domain, Edit your system /etc/hosts file with a an entry pointing the domain to one of our proxy servers:

##### Example /etc/hosts file
	##
	# Host Database
	#
	# localhost is used to configure the loopback interface
	# when the system is booting.  Do not change this entry.
	##
	127.0.0.1       localhost
	255.255.255.255 broadcasthost
	::1             localhost
 
	#proxy02.newmediadenver.com
	54.149.1.10 example.com www.example.com

Once this is saved, navigate to the domain in your browser. If the configuration is valid, you should see the site loading from our infrastructure, and non-www should redirect automatically to www.

**Important:** Remove the /etc/hosts entry immediately after you have completed verification to ensure your system is relying on real DNS records to load the domain. Failure to remove the modification will make it impossible to verify the site is properly configured when client DNS records are pointed.
Pointing DNS Records to Proxy Layer

*NOTICE! Do not perform this step until the proxy layer entry has been setup correctly!*

Below is an email template we send to clients. The same instructions apply to us if we are managing it:

##### Template Client Letter for DNS Record Changes
```
Hi {INSERT CLIENT},
One important step in a web development project is to effectively manage where your domain (e.g. www.yoursite.com) points. Failure to plan for this can result in timing and propagation delays.
To avoid any delays, SMB's hosting environments utilize a name based DNS, which allows us to point your domain at your current server up until the exact moment you’re ready to go live.
 
To get started, we ask you to update your DNS accordingly:
 
example.com (non-www version) A record to 54.149.1.10
www.example.com (www version) CNAME record to hosting.newmediadenver.com
 
Once updated, you will want to verify that this has been done correctly by using the following tools to check DNS records:
 
 
Check CNAME: http://mxtoolbox.com/CNAMELookup.aspx (input "www.example.com")
Check A record: http://www.mxtoolbox.com/DNSLookup.aspx (input "example.com")
 
Please note that some DNS changes can take up to anywhere from 24-48 hours to propagate.
Thanks,
{INSERT NAME} 
```
##### Verifying DNS Records

See previous section ("Pointing DNS Records to Proxy Layer") regarding the nslookup command tests for the non-www and www entries.

You can use the ops-verify-dns job in Jenkins to verify the DNS records as well: https://leroy.nmdev.us/view/OPS/job/ops-verify-dns

##### Mail Server Considerations

While many domain management interfaces will use MX to control mail separately, some sytems, like GoDaddy, rely on a "mail" CNAME. If the primary host is pointed elsewhere, the mail will be as well. To address this (at least within GoDaddy), remove the mail CNAME and add an A record for "mail" that will be pointed at the previous host IP. If there is a cPanel, that should also be updated to the mail host that was just created. Unless familiar with the unique system, it does not hurt to call the hosting entity and confirm settings.
