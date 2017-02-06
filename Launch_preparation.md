## Launch Preparation

In order to launch any site, there are a number of details and dates that must be gathered to ensure a smooth and expedient release to our production environment.  This document outlines the steps that must be completed before launch can be scheduled.

Within the JIRA project for the client, Project Managers must create a parent task to contain the necessary subtasks entitled 'Launch site _site name_'.  After creating this, there are two special subtasks which are necessary to add the complete information necessary to launch the site.

* _**Site Launch**_ - This is required for all projects, and must be completed no later than 5 business days before the scheduled launch.  This issue type contains fields for all the information necessary to launch the site.  Ensure that the ticket fields are filled out completely and accurately including the 'Due Date' which is used for scheduling purposes.
	* Consult with your developer to document any complexities which should be noted on the launch ticket
	* For non-standard launches (ie not on our infrastructure), please consult the developer for time estimate
		* As part of this please assure that:
			* We have all the launch credentials for the external host.
			* The client does not have an existing site on this hosting.
				* If they do we need explicit written permission to over write the site files and database that is already in place.
				* Please make sure the client makes a backup of the old site.
	* For standard launches (our infrastructure, no deployment complexities), please add the following time estimations to the launch ticket: drupal site ~30 min, wp site ~60 min
* _**SSL Certificate**_ - This is only required for sites using https.  This issue type contains fields for all the information required to set up the client's SSL.  The SSL ticket may be completed at any time before the launch date, and MUST be completed no more than 5 business days before the scheduled launch.

What happens next?

* Tickets of type SSL Certificate and Site Launch are surfaced on the Support Dashboard, in both the task list and on the Internal Support calendar.
* Internal support requests are reviewed daily, and the on call developer is expected to deploy any site falling on their support day (unless otherwise assigned).
