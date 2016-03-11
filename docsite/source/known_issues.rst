Known Issues and Troubleshooting
================================
The following are the current known issues.

PXE Booting on Dell Hardware
----------------------------
On rare occasions the Dell server will hang during the iPXE boot process. This can be verified by looking at the iDRAC console if the deployment seems to hang. The current known workaround is to simply rerun the Ansible ``dcaf-bada/site_deploy.yml`` playbook.

Pacemaker Issues During Deployment
----------------------------------
Due to an unknown bug at this time, Pacemaker will occasionally fail to configure properly during deployment. The current known workaround is to simply restart the deployment process, as this bug does not seem to be consistent.
