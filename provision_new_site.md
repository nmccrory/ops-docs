## Provision New Site

#### This process can be completed by anyone on the production team with access to Jenkins and an understanding of how to use it.

If a PM or PC is not comfortable completing this process or chooses not to access Jenkins, they can reach out to a production developer (through a ticket within the project) to request assistance in completing the task. If a production member is not available and other option have been exhausted, a support ticket can be submitted and someone will be assigned to help. Support tickets should be a last resort and will be placed back in the project once assigned.
In any case, a JIRA ticket MUST be created and eventually placed within the project. Time logged for spinup will be done so against the project.

#### Site Provisioning Video 
You can check out a video demonstrating this process:
Located in drud:  `drud file get 1fee/new-site-spinup.mp4`

####Run the jenkins-new-client job
Enter the required information to run the job:

* Click "build with parameters"
* sitename: this is the shorthand name of the client/site. It must be 13 characters or less. MUST BE IN ALL LOWERCASE
* apptype: select wp for WordPress or d7 for Drupal 7
* db_server_local: for WordPress, enter 127.0.0.1. For Drupal 7, leave as localhost
* theme: use sage for WordPress, use front.theme for Drupal 7
* theme_version: use master for WordPress, use 7.x-1.x for Drupal 7
* You will need the client's first name, last name, and email address. These will be used for client communication (IE: during outages).

Run the Job. When it has completed successfully, a notification will display in the __DevOps__ channel providing the staging and production URLs for the site.

The job will perform the following steps:

	* Create a databag for the site
	* Create a git repository in github for the site with an initial working codebase
	* Create DNS records for sitename.nmdev.us and sitenameprod.nmdev.us
	* Create staging-sitename and production-sitename jobs for the site in Jenkins
	* Adds staging and production proxy layer records
	* Runs initial create deployment on staging and production
	* Creates an initial backup of production
	
#### WordPress Only: Remove ___new_site___ flag from vault secret

After the jenkins-new-client job completes successfully, edit the site vault secret by running this command in iTerm: `drud secret edit databags/nmdhosting/<name of client database>` and remove the following line from all 3 environments:

`new_site: true`

#### Once this is completed, run the ___archive-sync___ job to copy the production backup down for local and staging.

Enter the required information to run the job:

* Click "build with parameters"
* Input "sitename" (Your lowercase project key)
* Confirm Direction is "Down"
* Then click "Build" 

#### WordPress Only: Adding theme license to databag
https://newmediadenver.atlassian.net/wiki/display/DEVOPS/Adding+Theme+License+Info+to+Site+Databag
