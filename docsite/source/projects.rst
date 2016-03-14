Projects
========

CSC DCAF includes the following CSC projects which are kept under version
control in a CSC GitHub repository:

- `Ansible-ScaleIO <https://github.com/csc/ansible-scaleio>`_ - The
  CSC Ansible-ScaleIO project, a fork of
  `sperreault/ansible-scaleio <https://github.com/sperreault/ansible-scaleio>`_,
  is used to install, configure and manage ScaleIO. For more information see the
  `csc/ansible-scaleio <https://github.com/csc/ansible-scaleio/wiki>`_
  wiki documentation. When used with BADA and Slimer, this project adds ScaleIO
  storage capabilities to the Red Hat OpenStack Platform.

- `Hanlon <https://github.com/csc/Hanlon>`_ - Hanlon is an advanced provisioning
  application which can deploy both bare-metal and virtual systems.

- `dcaf-abe (ABE) <https://github.com/csc/dcaf-abe>`_ - Project ABE creates the
  the AutoDeploy Node which is used as the basis for all CSC DCAF automation. It
  contains all the automation resources and is used to perform all automation
  tasks.

- `Slimer <https://github.com/csc/slimer>`_ - The CSC Slimer project, a fork of
  `abrezhnev/slimer <https://github.com/abrezhnev/slimer>`_, is used to deploy
  the Red Hat OpenStack Platform with high availability on base RHEL
  installations. For more information see the
  `csc/slimer <https://github.com/csc/slimer/wiki>`_ wiki documentation. When
  used with BADAthe result is a bare metal installation of the RHEL OS and Red
  Hat OpenStack Platform with high availability.

- `dcaf-bada (BADA) <https://github.com/csc/dcaf-bada>`_ - Project BADA provides
  an automated bare-metal deployment of the Red Hat Enterprise Linux OS. It can
  be used in tandem with Hanlon and Slimer to deploy a bare-metal installation
  of the RHEL OS with a Red Hat OpenStack Platform instance with high
  availability.

.. note::

    BADA is currently a private CSC project and requires a GitHub user account
    with access permissions.
