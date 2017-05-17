## Creating a New Site
_For the most part, creating a new site on leroy.drud.com is almost exactly the same as before._

- This process can be completed by anyone on the production team with access to Jenkins and an understanding of how to use it.
- If a PM or PC is not comfortable completing this process or chooses not to access Jenkins, they can reach out to a production developer (through a ticket within the project) to request assistance in completing the task.
  - If a production member is not available and other option have been exhausted, a support ticket can be submitted and someone will be assigned to help. Support tickets should be a last resort and will be placed back in the project once assigned. In any case, a JIRA ticket MUST be created and eventually placed within the project. Time logged for spinup will be done so against the project.

#### Site Provisioning Steps
- Run the [jenkins-new-client](https://leroy.drud.com/job/jenkins-new-client) job
- Enter the required information to run the job:
  - Click "Build with parameters"
      - `sitename`: this is the shorthand name of the client/site. It must be 13 characters or less. MUST BE IN ALL LOWERCASE
      - `apptype`: select wp for WordPress or d7 for Drupal 7
      - `db_server_local`: for WordPress, enter 127.0.0.1. For Drupal 7, leave as localhost
      - `theme`: use sage for WordPress, use front.theme for Drupal 7
      - `theme_version`: use master for WordPress, use 7.x-1.x for Drupal 7
      - `db_server_production`: select any of the entries
      - You will need the client's first name, last name, and email address. These will be used for client communication (i.e. during outages).
      - Run the Job.

The job will perform the following steps:
- Create a secret for the site
- Create a git repository in github for the site with an initial working codebase
- Create DNS records for sitename.drud.io and sitenameprod.drud.io
- Create staging-sitename and production-sitename jobs for the site in Jenkins
- Adds staging and production proxy layer records
- Runs initial create deployment on staging and production
- Creates an initial backup of production

#### WordPress Only: Remove `new_site` flag from vault secret
- After the jenkins-new-client job completes successfully, edit the site vault secret by running this command in iTerm: `drud secret edit databags/drudhosting/<name of client database>` and remove the following line from all 3 environments: `new_site: true`
- Once this is completed, run the [archive-sync](https://leroy.drud.com/job/archive-sync) job to copy the production backup down for local and staging.
- Enter the required information to run the job:
  - Click "build with parameters"
  - Input "sitename" (Your lowercase project key)
  - Confirm Direction is "Down"
  - Then click "Build"
WordPress Only: Adding theme license to databag
[Follow this link to add the theme license information to the site secret.](adding_theme_license_info.md)

## Where's My Stuff?
The big difference that you as a user will notice is the new location of the site's information.

To access a site's secret, the path is now `databags/drudhosting/{sitename}` (rather than `nmdhosting`)

To access a site's archive, you will still use `drud file`. To access an archive in the new system, the archive folder name has `drud-` prepended to it. 

To see some of these new site archives, run `drud file list drud-`

```bash
# Old syntax
drud file get {sitename}/{environment}-{sitename}-{timestamp}

# New Syntax
drud file get drud-{sitename}/{environment}-{sitename}-{timestamp}

# Example
drud file get drud-anewsite/production-anewsite-1493783396.tar.gz
```

The new jobs are at `https://leroy.drud.com/job/production-{sitename}` and `https://leroy.drud.com/job/staging-{sitename}`.

The new site URLs are at:

- https://{sitename}.drud.io
- https://{sitename}prod.drud.io

To use the local development drud binary, you will run all commands as normal except for the `drud dev add` command. When you want to develop against a site that's on the new infrastructure, you will add the `--secretDir` flag.

```
# Syntax
drud dev add {sitename} {environment} --secretDir="drudhosting"

# Example
drud dev add wpdemo production --secretDir="drudhosting"
```

_If you are developing a site against the old infrastructure, the flag is not necessary._
