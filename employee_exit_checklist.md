 From time to time it may be necessary to revoke employee access to our systems.  This document contains a checklist of steps, systems and services to address.

* Collect laptop.
* Reformat laptop.
* Rekey vault if user had unseal keys. In this case, consider auditing root tokens as well.
* Delete user from [gSuite](https://admin.google.com/drud.com/AdminHome?hl=en&pli=1&fral=1#UserList:org=3v377ch2rm5fsp). During the process, you will be asked to transfer all documents from Google Drive to a new owner and should do so.
* Remove user from [DRUD github](https://github.com/orgs/drud/people).
* Remove user from [newmedia github](https://github.com/orgs/newmediadenver/people) if applicable.
* Remove user and any of the user's bots from [slack](https://drud.slack.com/admin)
* Delete user's [Google Cloud Projects](https://console.cloud.google.com/iam-admin/projects?organizationId=722910822582).
* Remove user from each project in [GCP](https://console.cloud.google.com/iam-admin/projects?organizationId=722910822582).  Select all projects and use the info panel to remove the user's accounts.
* Rotate [Access Key ID and Secret Access Key](https://console.aws.amazon.com/iam/home#/security_credential) in the newmedia AWS console.
* Validate [AWS IAM users](https://console.aws.amazon.com/iam/home?#/users) for newmedia
* rotate passwords for service accounts.
* rotate github tokens for service accounts (newmediadenverbot, etc.)
* Revoke access to any lastpass secrets that are shared. Most should be invalidated instead. This is done in lastpass "sharing center".
* Make sure user is removed from [circle team](https://circleci.com/team). 

