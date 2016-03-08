DCAF Automation Quick Start Guide
=================================

This quick start guide describes how to use the following DCAF Automation projects:

- **dcaf-abe (ABE)** - ABE is used to provision a DCAF AutoDeployNode which is used as the basis for all DCAF automation. The AutoDeployNode contains all the automation resources and is used to perform all automation tasks.
- **Hanlon** - Hanlon is an advanced provisioning platform which can provision both bare-metal and virtual systems.
- **dcaf-bada (BADA)** - BADA provides an automated bare-metal deployment of the Red Hat Enterprise Linux OS using Hanlon.
- **Slimer** - Slimer is a fork of abrezhnev/slimer to deploy the Red Hat OpenStack Platform with high availability on BADA provisioned RHEL installations.
- **Ansible-ScaleIO** - Ansible-ScaleIO is a fork of sperreault/ansible-scaleio to install, configure and manage ScaleIO. ScaleIO provides additional storage capabilities to the Red Hat OpenStack Platform.

Before You Begin
================
Ensure that the following requirements are met:

User Access Requirements
------------------------
To retrieve the automation resources from their online repositories you will need the following:

- A valid github.com user account with access to the CSC Git repositories.
- A Red Hat user account with a valid subscription associated with it.

Network Requirements
--------------------
There are several network requirements for the deployment.

- DNS server IP addresses need to be provided
- NTP server IP address needs to be provided
- The following VLANS are required:
  - Out-of-band management (IPMI)
  - PXE Network
- Out-of-band Network IP address for each node
- Management IP address for each node.
- The AutoDeployNode has internet access and DNS

Physical Hardware
-----------------
BADA was developed and tested with the following hardware

- DELL PowerEdge 630 | PowerEdge 730
- DELL PERC H730 RAID Controller

Target Deployment Environment
-----------------------------
The target deployment environment consists of all nodes that will participate in the deployment.

- The physical hardware has been installed and connected to the network
- A 1GB connection for OOB management on the management switch
- 2 10GbE connections to the 10GbE network switches
- The upstream and management network have already been configured
- An on-site resource to configure the out-of-band management IP addresses.

Create the AutoDeployNode
=========================
The AutoDeployNode is the central location for all DCAF automation projects. Once an OS has been installed ABE will be used to provision and configure the rest of the DCAF Automation resources.

Install the RHEL OS
-------------------
Install the desired version of RHEL OS. This can be on a physical or virtual machine as long as all requirements are met. Be sure to set the hostname and static ip address.

For information on how to install Red Hat refer to the `Red Hat Install guide <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/sect-installation-graphical-mode-x86.html>`_

For more information on setting a static ip address refer to the `networking guide using the command line interface <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/sec-Using_the_Command_Line_Interface.html>`_.

Now that the OS has been installed it is time to stage the automation resources.

ABE - Configure the AutoDeployNode
----------------------------------
The remainder of the AutoDeployNode configuration is scripted and can be run after a few environment variables have been defined. The build script uses ABE to provision and configure the AutoDeployNode. Review the script for more details of all the steps which are performed.

Set the environment variables for Red Hat Subscription Manage credentials:
​

.. code-block:: bash

 export RHN_USER="username"
 export RHN_PASS="password"    # escape dollar signs (\$)
 export RHN_POOL="pool_id"     # 32-char pool ID

Create a ssh key file for GitHub access.  Put the text for a private key which has access to the GitHub repos in the lines below:

.. code-block:: bash

 cat << EOF > ~/github.pem
 -----BEGIN RSA PRIVATE KEY-----
 <insert_key_file_text_here>
 -----END RSA PRIVATE KEY-----
 EOF

Change the file permissions to ensure security.

.. code-block:: bash

 chmod 0600 ~/github.pem

With the environment variables defined and the ssh key file created, the build script can be launched:
​

.. code-block:: bash

 curl https://raw.githubusercontent.com/csc/dcaf-abe/master ansible/build.sh | bash​

.. note:: The build.sh script will perform a complete configuration of the AutoDeployNode using all project defaults. If there are changes required for your environment, a manual installation should be performed. Refer to the dcaf-ABE project documentation for more details.

At this point the AutoDeployNode has been deployed and is ready to start using for automation.

