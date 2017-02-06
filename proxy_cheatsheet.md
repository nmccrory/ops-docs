# Managing The Proxy Servers
---

To open the proxy server, run: `drud secret edit databags/nmdproxy/upstream`

## Flags:
Inside of each application block, any of the following optional flags can be specified

| Name | Type  | Description |
|------|-------|-------------|
| auth | Boolean | Places a basic HTTP authentication wall in front of the app |
| ssl_force | Boolean | Enforces a 301 redirect from http to https. Defining this flag by itself  |
| ssl_passthrough | Boolean | Used explicitly for passing traffic through to existing SSL sites |
| ssl | String | The name of the certificate secret in databags/nmdcerts/{env}/{name} |
| logging | Boolean | Enables access and error logging on both the proxy and web servers |

_Note: logs can be accessed by SSH-ing to web.newmediadenver.com, proxy.newmediadenver.com, web.nmdev.us and proxy.nmdev.us_

**Deprecated flags**:
www_force will now be ignored - the URL that is provided will be enforced.
e.g. 'www.newmedia.com' will respect requests for both newmedia.com and www.newmedia.com, whereas 'riotlabs.com' would only support traffic routed to riotlabs.com and would not serve www.riotlabs.com

**Common Entries**

```
production:
  webcluster01: # The name of the application pool
    apps: # A dictionary of web app definitions
      amlprod.nmdev.us: # Renders a site at https://amlprod.nmdev.us with an authentication wall in front of it and logging turned on
        auth: true # Enables the auth wall
        ssl: nmdev.us # Use a predefined SSL certificate
        ssl_force: true # Enforce SSL redirection
        logging: true # Enable logging
     
      # Renders a site at http://www.develops.guru
      www.develops.guru {}
    servers: # An array of servers that are responsible for rendering and returning the website itself
    - server web01.newmediadenver.com:80;
    - server web02.newmediadenver.com:80;
    - server web03.newmediadenver.com:80;
```      

--
### Verifying a Proxy Entry
To verify the proxy configuration is correct prior to pointing the client's domain, Edit your system /etc/hosts file with a an entry pointing the domain to one of our proxy servers:

```
**Example /etc/hosts file**
	##
	# Host Database
	#
	# localhost is used to configure the loopback interface
	# when the system is booting.  Do not change this entry.
	##
	127.0.0.1       localhost
	255.255.255.255 broadcasthost
	::1             localhost
 
	54.149.1.10 example.com www.example.com
```
Once this is saved, navigate to the domain in your browser. If the configuration is valid, you should see the site loading from our infrastructure, and non-www should redirect automatically to www.

**Important:** Remove the /etc/hosts entry immediately after you have completed verification to ensure your system is relying on real DNS records to load the domain. Failure to remove the modification will make it impossible to verify the site is properly configured when client DNS records are pointed.
Pointing DNS Records to Proxy Layer