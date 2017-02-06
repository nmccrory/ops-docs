## Migrating/Integrating an Existing Site Internally

### Notice: Only submit tickets for migrating existing sites or custom projects.

All new projects follow the Provision New Site process.

#### Motivation
While many projects that newmedia or 1FEE will work on are conceived by our teams, we do also take on projects pertaining to existing sites for various ongoing development requirements. In these situations, we need to make sure we can work on the existing project within our infrastructure and development environment. This requires taking the existing site's codebase, database, and assets (images, pdfs, etc) and importing them into our required format. As this is not a trivial task compared to provisioning a new project from scratch in our infrastructure, a Support ticket should be created to facilitate importing the existing site to our infrastructure.

### Instructions for migrating/integrating existing sites or custom projects
1. Follow the Provision New Site as if you were starting a new project.
2. Log in to JIRA and create a new task under the newly-created project using the following structure: Migrate CLIENTPROJECT into our infrastructure.
3. Add a time estimate:
	a. Existing Drupal or Wordpress site: 6 hours.
	b. Custom Application: 16 hours.
4. Add a due date
5. Assign the ticket to the appropriate/selected resource on or before the sprint planning meeting.
6. In the description, provide the following information:
	a. Current Website URL
	b. Current Website Platform (Drupal, Wordpress, Other (specify))
	c. Are we moving the production site to our hosting environment? (Yes/No)
	d. How to access the current codebase (store credentials in drud secret and reference from the ticket)
	e. How to get a current backup of the site, including database and uploaded assets.
