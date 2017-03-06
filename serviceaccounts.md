# Service Accounts in use by ops

## Github

* drud-test-machine-account: Used for builds, for vault access. Parties involved have this in lastpass. Expected to be only used in the drud organization.
* newmediadenverbot: Hyper-powerful account (admin) used for jenkins activities for hosting.
* drud-monitoring-machine-account: Not currently used (3/6/2017). Had been intended for vault monitoring.
* drud-test-user: Not currently used (3/6/2017), can be used for manual testing.

## Dockerhub

* druddockerpullaccount: Used in builds to allow pulling from a parent private repo. Parties involved have the login info in lastpass. It should have only read privs.

## Vault.drud.com

* "monitoring" token, which is renewed automatically by the drud-prod jenkins job. It has read access only to the /secret/monitoring/ secret which is used for stackdriver monitoring. More details in [vault playbook](https://github.com/drud/ops-docs/blob/master/vault_playbook.md)
