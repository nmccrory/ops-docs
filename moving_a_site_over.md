# Moving a site to the new infrastructure (using connected VPCs)

## Overview
- [Begin the content freeze](#begin-the-content-freeze)
- [Run a backup on https://leroy.nmdev.us/job/production-{site}](#run-a-backup)
- [Run jenkins-transition-site](#preparing-for-an-existing-site)
  - Creates new jobs and secrets
- [Copy the S3 archive over from `{sitename}/` to `drud-{sitename}/`](#copy-the-site-archive)
- [Copy the `nmdhosting` secret over to `drudhosting` and modify a few values](#copy-the-site-metadata)
- [Modify the proxy layer to move the production URL from `webcluster01` to `drud-elb`](#move-the-production-url-from-webcluster01-to-drud-elb)
- [Run a create on https://leroy.drud.com/job/production-{site}](#deploy-it)
  - The site is now live on the new infra, but nothing is pointing to it
- [Confirm that the site is working in the new infra using /etc/hosts](#checking-to-make-sure-everything-worked)
- [Run https://leroy.nmdev.us/job/ops-deploy-by-url for the production URL you just moved **NOTE THE NMDEV.US!!!**](#forward-the-traffic)
  - This will set drud-elb as a downstream on the old proxy servers and the site's code/DB will now be living in the new infra
- [End the content freeze](#end-the-content-freeze)
- [Repoint the DNS](#point-the-dns-at-drud)

## Begin the content freeze
All changes to the database and asset files from now until the content freeze is over will be lost.

## Run a backup
Run https://leroy.nmdev.us/job/production-{site}

## Preparing for an existing site
Run https://leroy.drud.com/job/jenkins-transition-site

This performs the following steps:
- Create the Jenkins jobs for deploying the site
- Create the default DNS entries for the site
- Generate a new proxy entry for staging/production

## Copy the site archive
- If you use drud file, you will need to download each archive locally, then upload it back to S3, if that's still the way you want to go:
  - Find the most recent production archive with `drud file list {sit}ename}/production` (note - this is the file with the largest number/timestamp)
  - Pull the archive down with `drud file get {sitename}/production-{sitename}-{timestamp}.tar.gz`
  - Upload the archive to the new space with `drud file put production-{sitename}-{timestamp}.tar.gz drud-{sitename}/production-{sitename}-{timestamp}.tar.gz`
- Install [mc](https://github.com/minio/mc) with `brew install minio/stable/mc`
  - mc allows for direct copying from one S3 location to another directly
  - Configure mc with `mc config host add nmd https://s3.amazonaws.com {AWS_ACCESS_KEY_ID} {AWS_SECRET_KEY_ID} S3v4` replacing the AWS keys with the `s3fileperms_aws_access_key_id` and `s3fileperms_aws_secret_access_key` with values from admin/sites/amazon.com.
      - You will see another `S3` key in there. Ignore it. This key has the same permissions as the `drud file` tool and will be removed after the migrations are done (since the key may be diluted)
  - Find the most recent archive from the nmd volume: `mc ls nmd/nmdarchive/{sitename}`
  - Copy the archive from the old location to the new location: `mc cp nmd/nmdarchive/{sitename}/production-{sitename}-{timestamp}.tar.gz nmd/nmdarchive/drud-{sitename}/production-{sitename}-{timestamp}.tar.gz`

## Copy the site metadata
- Get the old site information `drud secret read databags/nmdhosting/{sitename}`
- Open the new secret for writing with `drud secret write databags/drudhosting/{sitename}` and copy the data from the old secret. Make the following changes before closing/saving:

Change the db host in production and staging. Production can be set to `db2.production`, `db3.production` or `db4.production`. Staging must be set to `db1.staging`.

```
production:
-  db_host: mysql.newmediadenver.com
+  db_host: db2.production
staging:
-  db_host: percona.nmdev.us
+  db_host: db1.staging
```

Remove the `db_port` and `custom_port` lines from the production section

```
production:
-  db_port: "3307"
-  custom_port: enabled
```

Replace all instances of `.nmdev.us` with `.drud.io`. This should be in the `url` and the `server_aliases` fields. *_Note:_ If the production URL is not set to a `nmdev.us` URL, then leave it alone.

```
production:
   server_aliases:
-  - drudioprod.nmdev.us
+  - drudioprod.drud.io

-  url: http://drudioprod.nmdev.us
+  url: http://drudioprod.drud.io

staging:
   server_aliases:
-  - drudio.nmdev.us
+  - drudio.drud.io

-  url: http://drudio.nmdev.us
+  url: http://drudio.drud.io
```

#### For WP Sites
Add a `search_replace` entry for the old site URLs:

```
production
  search_replace:
  - http://drudioprod.nmdev.us

staging:
  search_replace:
  - http://drudio.nmdev.us
```

## Move the production URL from `webcluster01` to `drud-elb`
Open the proxy layer (at `databags/nmdproxy/upstream`) to move the production URL application from `webcluster01` to `drud-elb`.

## Deploy it!
- Run a `create` on https://leroy.drud.com/job/production-{sitename}/
  - When this finishes, the site will be live, but nothing is pointed to it
- Run a [dev-sync](https://leroy.drud.com/job/dev-sync) (backup production, copy archives to staging/_default, create staging)

## Checking to make sure everything worked
TL;DR: Edit /etc/hosts (you will no longer be using the `54.149.1.10` IP): `35.166.147.204 drud.com www.drud.com`

### Verify and Troubleshoot
**Verify that your changes worked** using /etc/hosts by following the steps in [Verifying a Proxy Entry](proxy_cheatsheet.md#verifying-a-proxy-entry)

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

- The jenkins create job (in leroy.drud.com) ran successfully
- The `databags/drudhosting/SITENAME` production `server_alias` and `url` values match the proxy entry app name in `databags/nmdproxy/upstream`
- If the job failed at the Let's Encrypt step, read through [Let's Encrypt](lets_encrypt.md) for a sanity check.

## Forward the Traffic
Run this job on the new infrastructure: [ops-deploy-by-url](https://leroy.drud.com/job/ops-deploy-by-url/) with `Environment = production` and `URL = the URL that you just set above (e.g. www.careincommon.com)`. Make sure the URL value does NOT contain http(s)://

## End the Content Freeze

## Point the DNS at DRUD
Clients hosting in DRUD infrastructure require the following DNS records:

Name/Host/Alias | Time to Live (TTL) | Record Type | Value/Answer/Destination
------------ | ------------- | ----------- | ------------------------
Blank or @ | 300 | A | 34.210.3.109 <br> 35.166.147.204
www | 300 | CNAME | hosting.drud.com

Any client that hosts their site within our infrastructure must have the appropriate DNS records pointed at our proxy layer in AWS. In laymen's terms, they need to point their sites (i.e. clientsite.com and www.clientsite.com) at our infrastructure. That gives us the ability to manage which server(s) the domain points to without involving the client, which is highly beneficial for a variety of reasons (efficiency, removing the client as a dependency, etc).

## Run a back-up on the old project in leroy.nmdev.us/job/production-sitename
This will capture any differences during the back-up/create

## Delete the DNS for the sitename.nmdev.us URLs
Run leroy.nmdev.us/job/ops-delete-dns with `sitename` as the key.

## Disable the old Jenkins job
Go into the Jenkins job and disable it. Delete the build steps. Paste the link to the new Jenkins job.
