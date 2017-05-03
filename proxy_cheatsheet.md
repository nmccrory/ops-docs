# Managing The Proxy Servers
---

To open the proxy server, run: `drud secret edit databags/nmdproxy/upstream`

## Flags:
Inside of each application block, any of the following optional flags can be specified

| Name | Type  | Description | Example
|------|-------|-------------|---------
| `auth` | Boolean | :lock: Places a basic HTTP authentication wall in front of the app | `auth: true`
| `ssl_force` | Boolean | Enforces a 301 redirect from http to https. Defining this flag by itself  | `ssl_force: true`
| `ssl` | String | The name of the certificate secret in databags/nmdcerts/{env}/{name} | `ssl: "newmedia.com"`
| `logging` | Boolean | :page_with_curl: Enables access and error logging on both the proxy and web servers | `logging: true`
| `auth_exclude` | String array | An array of paths where the basic authentication wall should not be present. These paths are unpacked as [nginx location directives](https://www.digitalocean.com/community/tutorials/understanding-nginx-server-and-location-block-selection-algorithms#matching-location-blocks) | (Single line example, see multiline in the example below) - `auth_exclude: { '/not-auth', '/api', '/dumb_module_path' }`
| `verify_ssl` | Boolean | Run SSL verification commands against the installed SSL certificate | `verify_ssl: true`
| `proxy_buffer_size` | string | Controls the proxy buffer size |  `proxy_buffer_size: "25m"`
| `proxy_buffers` | string | Controls the proxy buffers | `proxy_buffers: "8 25m"`
| `proxy_busy_buffers_size` | string | Controls the proxy busy buffers size | `proxy_busy_buffers_size: "25m"`
| `client_max_body_size` | string | Controls the client max body size | `client_max_body_size: "25m"`
| `maintenance` | Boolean | Redirects all requests to the [/var/www/maintenance.html](https://github.com/drud/drudfab/blob/master/roles/manage_proxy/files/maintenance.html) page on the proxy server and returns a 503 error code | `maintenance: true`

_Note: logs can be accessed by running the [ops-tail-server-log](https://leroy.drud.com/job/ops-tail-server-log/build?delay=0sec) Jenkins job_

**Deprecated flags**:
www_force will now be ignored. Instead, the application name that is provided here (the url) will be enforced. e.g. `www.newmedia.com` would serve requests for both `newmedia.com` and `www.newmedia.com`, whereas `riotlabs.com` would only support traffic routed to `riotlabs.com` and would not respond to requests to `www.riotlabs.com`

**Common Entries**

```yaml
production:
  drud-elb: # The name of the application pool
    apps: # A dictionary of web app definitions
      amlprod.drud.io: # Renders a site at https://amlprod.nmdev.us with an authentication wall in front of it and logging turned on
        auth: true # Enables the auth wall
        ssl_force: true # Enforce SSL redirection
        logging: true # Enable logging

      www.develops.guru: {} # Renders a site at http://www.develops.guru

      christestprod.drud.io:
        ssl_force: true
        verify_ssl: true # Runs certificate verification steps to ensure that the certificate is valid before deploying it
        auth_exclude: # An array of nginx location directives
        - "/no-auth" # Removes authentication on all /no-auth/* paths
        - "= /exact" # Removes authentication from /exact path only, no sub paths

      www.listenfordns.ninja: # Renders a site at https://www.listenfordns.ninja with a Let's Encrypt SSL certificate
        ssl_force: true # When specified without the ssl or ssl_passthrough directive, this will apply for a Let's Encrypt certificate.

    servers: # An array of servers that are responsible for rendering and returning the website itself
    - server drud-elb.production:80;
```

--
### Verifying a Proxy Entry
To verify the proxy configuration is correct prior to pointing the client's domain, Edit your system /etc/hosts file with a an entry pointing the domain to one of our proxy servers:

**Example /etc/hosts file**

```
	##
	# Host Database
	#
	# localhost is used to configure the loopback interface
	# when the system is booting.  Do not change this entry.
	##
	127.0.0.1       localhost
	255.255.255.255 broadcasthost
	::1             localhost

	35.166.147.204 example.com www.example.com
```
Once this is saved, navigate to the domain in your browser. If the configuration is valid, you should see the site loading from our infrastructure, and non-www should redirect automatically to www.

**Important:** Remove the /etc/hosts entry immediately after you have completed verification to ensure your system is relying on real DNS records to load the domain. Failure to remove the modification will make it impossible to verify the site is properly configured when client DNS records are repointed.

## Some Helpful Jenkins Jobs
**These jobs only run the proxy portion of the drudfab deployment.** If you've made a change in databags/drudhosting/SITENAME, chances are you cannot use this job. If however you just edited the proxy layer and no changes need to be made on the web server itself, (i.e. changing the web server's vhost file to accept a new url, adding wp rewrite entries, etc)

- [https://leroy.drud.com/job/ops-deploy-by-url](https://leroy.drud.com/job/ops-deploy-by-url/build?delay=0sec) - Updates a sinlge site's configuration given a URL. _This URL cannot have http or https in it._

