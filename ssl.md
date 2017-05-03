## Let's Encrypt Certificates
### About
Let's Encrypt is a free service that allows us to generate our own secure SSL certificates. Clients can still buy their own certificate and it can still be implemented using the existing documentation, or we can provide them with a certificate that costs us nothing.

### Verify DNS
Start by running this Jenkins job: [ops-verify-dns](https://leroy.drud.com/job/ops-verify-dns)

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

where the job is claiming that the DNS records are not configured correctly, then you are either working with a domain that has either been mis-configured or you are working with a site that has not yet been pointed at our domain.

Ensure the records are correct:

Name/Host/Alias | Time to Live (TTL) | Record Type | Value/Answer/Destination
------------ | ------------- | ----------- | ------------------------
Blank or @ | 300 | A | 35.166.147.204 <br> 34.210.3.109
www | 300 | CNAME | hosting.drud.com

For more information, see [Point the DNS at DRUD](launch_a_site.md#point-the-dns-at-drud).


### Get the Certificate
Once the DNS is pointed at DRUD, this is the section for you!

If you had to setup a redirect/new upstream, ensure you delete the entire application block you created as part of [Forwarding Traffic to an Existing Server](launch_a_site.md#forwarding-traffic-to-an-existing-server).

Create the production proxy entry. Set `ssl_force = True`, and leave `ssl` undefined.

```yaml
production:
  webcluster01:
    apps:
      www.careincommon.com:
        ssl_force: true
```


The next time an update runs on the site, the certificate will automatically be applied for, added to the acmetool certificate renewal scheduler queue, and connected to the site. You should never have to worry about it ever again.

:tada:

## Pre-defined SSL Certificates
To add an SSL certificate to a production site, run:

```bash
drud secret write databags/nmdproxy/certs/production/<SITENAME.com>
#example:
drud secret write databags/nmdproxy/certs/production/exploreyourspirit.com
```
Replacing <SITENAME> with the URL of your site without the http(s)://www.
If the secret already exists, you should use 'edit' rather than 'write'.

You can see examples of other certificates by running:

```bash
$  drud secret list databags/nmdproxy/certs/production
keys:
- 1fee.com
- accuraterealtyaa.com
- aiplife.com
- attorneycharliehall.com
- beautehair.co
- careincommon.com
...
```
Then you can pick a certificate from that list (I'll choose one at random):
`$  drud secret read databags/nmdproxy/certs/production/newmediadenver.com`

You'll see 3 sections. The 'ca' field, the 'crt' field and the 'key' field.

- The 'ca' field should be populated with the Certificate Authority bundle file. (Filename contains the text "ca", "ca-bundle", "IntermediateCA", etc.) 
- The 'crt' field should be populated with the Certificate file. (Usually, this file ends in *.crt)
- The 'key' field should be populated with the Private key file (Usually, this file ends in *.key)

When the CSR file was generated (the file that was uploaded to the SSL provider), a KEY file was created at the same time at the same path that the CSR file was created. So if the site's CSR file was created in your Downloads directory, the KEY file should be in the Downloads directory as well.

**There's one YAML gotcha that you should be aware of:** In order to do multi-line secrets, you need to put this ` |-` right after your paramater.
So your secret will look akin to this (just longer):

___Dots denote spaces___

```
ca: |-
..-----BEGIN CERTIFICATE-----
..MIIE0DCCA7igAwIBAgIBBzANBgkqhkiG9w0BAQsFADCBgzELMAkGA1UEBhMCVVMx
..EDAOBgNVBAgTB0FyaXpvbmExEzARBgNVBAcTClNjb3R0c2RhbGUxGjAYBgNVBAoT
..GIo/ikGQI31bS/6kA1ibRrLDYGCD+H1QQc7CoZDDu+8CL9IVVO5EFdkKrqeKM+2x
..LXY2JtwE65/3YR8V3Idv7kaWKK2hJn0KCacuBKONvPi8BDAB
..-----END CERTIFICATE-----
..-----BEGIN CERTIFICATE-----
..MIIEfTCCA2WgAwIBAgIDG+cVMA0GCSqGSIb3DQEBCwUAMGMxCzAJBgNVBAYTAlVT
..MSEwHwYDVQQKExhUaGUgR28gRGFkZHkgR3JvdXAsIEluYy4xMTAvBgNVBAsTKEdv
..KHAN4v6mF56ED71XcLNa6R+ghlO773z/aQvgSMO3kwvIClTErF0UZzdsyqUvMQg3
..qm5vjLyb4lddJIGvl5echK1srDdMZvNhkREg5L4wn3qkKQmw4TRfZHcYQFHfjDCm
..rw==
..-----END CERTIFICATE-----
..-----BEGIN CERTIFICATE-----
..MIIEADCCAuigAwIBAgIBADANBgkqhkiG9w0BAQUFADBjMQswCQYDVQQGEwJVUzEh
..MB8GA1UEChMYVGhlIEdvIERhZGR5IEdyb3VwLCBJbmMuMTEwLwYDVQQLEyhHbyBE
..HmyW74cNxA9hi63ugyuV+I6ShHI56yDqg+2DzZduCLzrTia2cyvk0/ZM/iZx4mER
..dEr/VxqHD3VILs9RaRegAhJhldXRQLIQTO7ErBBDpqWeCtWVYpoNz4iCxTIM5Cuf
..ReYNnyicsbkqWletNw+vHX/bvZ8=
..-----END CERTIFICATE-----
crt: |-
..-----BEGIN CERTIFICATE-----
..MIIFNzCCBB+gAwIBAgIIGjsFEkp5+PkwDQYJKoZIhvcNAQELBQAwgbQxCzAJBgNV
..BAYTAlVTMRAwDgYDVQQIEwdBcml6b25hMRMwEQYDVQQHEwpTY290dHNkYWxlMRow
..D6PpkitNAqNZjuKw26lMEAlN2hA+t5zyc02gedJZfjGV3dNYGSDLhy1zVOp9a62M
..n9Bd7zt59K8xyvFxeuMB4ysQPLnAtao73i5R4P2JHJfoYVhrGNc2m+RdLw==
..-----END CERTIFICATE-----
key: |-
..-----BEGIN PRIVATE KEY-----
..MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQCuVgTxOV9FFsaI
..8nbpBQ4fxWen6q/oio3vngwZd+4kx5TeOxD1HgLtDimqaaBAIGrY7YJ0OdXYkw+E
..cFhMMSMmDxux09mt3LwNVBriwtgJ80eD/AeZo24n2z5czyvvAmQDO+y/OS/Zej48
../dYDhCXs+UA76sgzUM/+e/s=
..-----END PRIVATE KEY-----
```

When you are done, close the secret, then upload all of these files to the drud file tool.
`drud file put <filename>.zip <project>/<filename>.zip`
