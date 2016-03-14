AutoDeployNode Creation
=======================

The AutoDeployNode is our central location for running Ansible playbooks. Since our projects have multiple dependencies we have automated the process to install and configure the AutoDeployNode as much as possible.

Install the RHEL OS
-------------------

The first step is to install RHEL.  This can be physical or virtual as long as the requirements are met.

#. Using the media that is appropriate for your hardware boot into the RHEL installation.
#. On the installation summary page, you may see different selections with yellow exclamation or warning marks. These are areas that require some setup:

- Date & Time : Current Date / Time ? Time Zone
- Installation Source : Local Media
- Software Selection : Minimal Install
- Installation Destination : Partitioning : Automatically configure partitioning
- Network & Hostname : Enable the network interface and configure with relative static network information
- Root Password : Set the root password
- Create User: autodeploy

Here is a link to the `Red Hat Install guide <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/sect-installation-graphical-mode-x86.html>`_

Set the Static IP
~~~~~~~~~~~~~~~~~

Boot the AutoDeployNode and configure the network interface with a static IP address.  For more information refer to the `networking guide using the command line interface <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/sec-Using_the_Command_Line_Interface.html>`_.


.. code-block:: bash

    vi /etc/sysconfig/network-scripts/ifcfg-<nic>

Modify the file to resemble this configuration with your specific network configuration.

.. code-block:: bash

    BOOTPROTO="none"
    ONBOOT=yes
    IPADDR=x.x.x.x
    NETMASK=x.x.x.x
    GATEWAY=x.x.x.x
    DNS1=x.x.x.x

Set the Hostname
~~~~~~~~~~~~~~~~~

Next configure the hostname:

.. code-block:: bash

    hostnamectl set-hostname autodeploy.local

Now that the OS has been installed it is time to stage the automation resources.

Configure AutoDeployNode
------------------------

Use Scripted Install
~~~~~~~~~~~~~~~~~~~~

The build of the AutoDeployNode is scripted and can be run after a few environment variables have been defined.  The AutoDeployNode will need to install packages and download files needed by RHEL so it needs to be registered with subscription-manager.  Environment variables for Red Hat Subscription credentials should be defined:

.. code-block:: bash

    export RHN_USER="username"
    export RHN_PASS="password" # escape dollar signs (\\$)
    export RHN_POOL="pool_id" # 32-char pool ID

Next, an ssh key file should be created for GitHub access. Put the text for a private key which has access to the GitHub repos in the lines below:

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

.. code-block:: bash

    curl https://raw.githubusercontent.com/csc/dcaf-abe/master/ansible/build.sh | bash

Review the details in the build script for a description of all the steps which are performed to build the AutoDeployNode.

.. note:: The :code:`build.sh` script will perform a complete install and configuration of the AutoDeployNode using all project defaults. If there are changes required for your environment, a manual installation should be performed.


Manual Install
~~~~~~~~~~~~~~

The AutoDeployNode will need to install packages and download files needed by RHEL so it needs to be registered with subscription-manager.

Most commands require elevated privileges so you may need to :code:`su -`.  Register with Red Hat Subscription Manager. Fill in the username and password with credentials that have a valid Red Hat subscription associated with it.

.. code-block:: bash

    su -
    subscription-manager register --username=your_user --password=your_password

Find one of the repositories that include "Red Hat Openstack". Once a
subscription is found that provides Openstack note the "Pool ID"

.. code-block:: bash

    subscription-manager list --all --available
    subscription-manager attach --pool="Pool ID"

Disable all repositories, then enable RPM repositories as needed.

.. code-block:: bash

    subscription-manager repos --disable=*
    subscription-manager repos --enable=rhel-7-server-rpms \
    --enable=rhel-7-server-optional-rpms \
    --enable=rhel-7-server-extras-rpms \
    --enable=rhel-7-server-openstack-6.0-rpms \
    --enable=rhel-server-rhscl-7-rpms \
    --enable=rhel-ha-for-rhel-7-server-rpms

Next install the required support packages; epel-release, git and wget.

.. code-block:: bash

    yum -y install https://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm
    yum -y install git wget

.. note:: Ansible v2.0 is not available from EPEL and must be installed from source until EPEL availability.

To build an Ansible RPM from source, additional packages are required:

.. code-block:: bash

    yum -y install rpm-build make asciidoc python2-devel python-setuptools

Now the source for Ansible must be cloned. A particular version of Ansible is currently tested and supported for use, as indicated below. The new RPM is installed as well as additional Ansible dependencies.

.. code-block:: bash

    git clone git://github.com/ansible/ansible.git --recursive
    cd ansible/
    git checkout v2.0.1.0-1
    git submodule update --init --recursive
    make rpm
    yum -y --nogpgcheck localinstall ./rpm-build/ansible-*.noarch.rpm
    cd ..


**Retrieve the CSC DCAF projects**

Ansible has been installed and will be used to perform an automated download of the CSC DCAF project resources. First we need to download the :code:`initial_stage` play from the :code:`dcaf-abe` Git repository.

.. code-block:: bash

    wget https://raw.githubusercontent.com/csc/dcaf-abe/master/ansible/initial_stage.yml

In order to download the projects from GitHub, a keyfile must be created. Put the text for a private key which has access to the GitHub repos in the lines below:

.. code-block:: bash

    cat << EOF > ~/github.pem
    -----BEGIN RSA PRIVATE KEY-----
    <insert_key_file_text_here>
    -----END RSA PRIVATE KEY-----
    EOF

Change the file permissions to ensure security.

.. code-block:: bash

    chmod 0600 ~/github.pem

Now the initial\_stage.yml playbook can be run, as shown below:

.. code-block:: bash

    ansible-playbook initial_stage.yml --extra-vars "github_key_file=~/github.pem"

Now that the ABE project has been retrieved it can be used to install the remaining support packages. Change into the ABE project directory.

.. code-block:: bash

    cd /opt/autodeploy/projects/dcaf-abe/ansible

Next run the :code:`stage_resources.yml` play to download the CSC DCAF automation resources. The :code:`stage_resources.yml` play requires valid user accounts to GitHub and Red Hat as outlined in section 2.1 User Access Requirements. Before you run the play change into the :code:`/opt/autodeploy/projects/dcaf-abe/ansible` directory and edit the following variables in the :code:`inventory/group_vars/all.yml` file.

.. code-block:: yaml

    # Required User Variables
    rhn_user:
    rhn_pass:
    github_key_file: (location of the github.pem file created above)

Run the stage\_resoures.yml play:

.. code-block:: bash

    ansible-playbook stage_resources.yml

Configure ABE variables
~~~~~~~~~~~~~~~~~~~~~~~

The ABE project contains the automation resources to complete the installation and configuration of the AutoDeployNode. It uses Ansible for all automation. Before any playbooks can be run, the Ansible configuration variables for ABE need to be edited per your environment.  Configure ABE accordingly by editing the variables in the :code:`/opt/autodeploy/projects/dcaf-abe/ansible/inventory/group_vars/all.yml`.

.. code-block:: yaml

    use_scaleio:
    use_dcaf_bada:

There are basic variables that apply to all deployments that will need to be modified before deployment.

By default, the DHCP server will be installed with the following configuration:

.. code-block:: yaml

    dns1: 8.8.8.8
    dhcp_start: 20
    dhcp_end: 60

The DHCP start and end values above are the last octet of the subnet the server is installed in. For example,

172.17.16.20 would be :code:`dhcp_start: 20` and 172.17.16.60 would be :code:`dhcp_end: 60`.

To use alternate values, add the above three lines to the :code:`dcaf-abe/ansible/inventory/group_vars\all.yml` file with your own values.

Running ABE Playbooks
~~~~~~~~~~~~~~~~~~~~~

Now that the ABE variables have been set use the ABE project to finish the AutoDeployNode deployment.

.. code-block:: bash

    cd /opt/autodeploy/projects/dcaf-abe/ansible
    ansible-playbook main.yml

The :code:`main.yml` playbook will also run the :code:`site_docker.yml` and :code:`site_discovery.yml` playbooks.

The :code:`site_docker.yml` playbook will start the Hanlon Docker environment. First it will clean up any existing containers. Then it will start the Mongo, Hanlon Server and TFTP Server containers.

The :code:`site_discovery.yml` playbook will configure the DHCP service and prepare the Hanlon Server for the bare metal BADA deployment.

At this point the AutoDeployNode has been deployed and is ready to start using for automation.
