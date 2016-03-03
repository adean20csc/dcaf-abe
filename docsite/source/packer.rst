Introduction
------------

The Packer-based build process of dcaf-abe provides an automated method
of building an all-encompasing qcow2 image.

Requirements
------------

1. Recent Linux distribution (tested on CentOS 7.1, RHEL 7.1 and Fedora
   22)
2. VT or AMD-V processor

-  If you are running this in a virtual machine the hypervisor must
   support nested virtualization

3. Docker
4. RHEL 7.1 ISO
5. At least 25 GB disk space
6. qemu-img (if the qcow2 image needs to be converted to raw)

Packer Container
----------------

Contents
~~~~~~~~

The container has been pre-installed with the necessary software to
execute the build process. - Packer (Latest tagged release) -
``docker-entrypoint.py`` (Python and required libraries) - QEMU - which
is used to create KVM-based virtual machine (required and used by Packer
to automate the build process)

Build Process
~~~~~~~~~~~~~

The following steps will build the container if you do not use the one
that should be available on the Docker Registry.

1. ``git clone https://github.com/csc/dcaf-abe; cd dcaf-abe`` - Clone
   project from github and change directory to dcaf-abe.
2. ``docker build -t dcaf-abe .`` - build container.

Once the container is built we can continue with using Packer to build
the qcow2 image of the dcaf-abe node.

Container Execution and Image Build
-----------------------------------

Prerequisites
~~~~~~~~~~~~~

1. ``mkdir -p path/os`` - Create a folder where the RHEL 7.1 ISO will be
   available for use in the container and Packer.
2. ``cp rhel-server-7.1-x86_64-dvd.iso path/os`` - Copy the RHEL 7.1 DVD
   as-is.
3. ``chcon -Rt svirt_sandbox_file_t path`` - Set the SELinux context on
   the directory.

Docker execution arguments
~~~~~~~~~~~~~~~~~~~~~~~~~~

To execute a virtual machine in the container the following required
options need to be set: - `Privileged
mode <https://docs.docker.com/reference/run/#runtime-privilege-linux-capabilities-and-lxc-configuration>`__
(``--privileged=true``) - `Docker
Volumes <https://docs.docker.com/userguide/dockervolumes/>`__ - KVM
device (``-v /dev/kvm:/dev/kvm:rw``) - TUN device
(``-v /dev/net/tun:/dev/net/tun:rw``) - Build directory
(``-v path:/build dcaf-abe-packer-build``) - This is your location on
the filesystem where the qcow2 output will be available and where Packer
will retrieve the RHEL ISO specifically
``path/os/rhel-server-7.1-x86_64-dvd.iso``

There are two optional parameters: - `Expose
Ports <https://docs.docker.com/reference/run/#expose-incoming-ports>`__
- The option ``-p 5900:5900`` performs an iptables NAT operation to
allow connection to VNC if you wish to watch the RHEL installation
process. - `Clean
up <https://docs.docker.com/reference/run/#clean-up-rm>`__ - ``--rm``
deletes the container after Packer is complete.

Entrypoint arguments
^^^^^^^^^^^^^^^^^^^^

| The ``--username`` and ``--password`` options are used for Red Hat
  Subscription Manage.
| The ``--packer`` option is the command line arguments that need to be
  passed to execute a proper build.

**NOTE:** Currently this should only be
``--packer "build /opt/dcaf-abe/rhel7.json"``

**NOTE:** Do not connect via VNC until Packer has outputted
``Waiting for SSH to become available...`` otherwise the build will
fail.

Example
^^^^^^^

.. code:: bash

    docker run \
    --rm \
    -it \
    -e HOME=/ \
    --privileged=true \
    -p 5900:5900 \
    -v /dev/kvm:/dev/kvm:rw \
    -v /dev/net/tun:/dev/net/tun:rw \
    -v path:/build \
    dcaf-abe \
    --username REDACTED \
    --password REDACTED \
    --packer "build /opt/dcaf-abe/rhel7.json"

Transferring the image
----------------------

Once the image has been created it will be available
``path/output/packer-dcaf-abe-qcow2``. This image can be converted to
any format that ``qemu-img`` supports, in our example we will be using
raw.

Convert to RAW
~~~~~~~~~~~~~~

Before getting started lets install ``qemu-img`` via
``yum install qemu-img -y``

.. code:: bash

    qemu-img convert -O raw packer-dcaf-abe-qcow2 dcaf-abe.raw

Process for physical hardware
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Below is an example of ``dd`` required command line options and
execution. First you will need to run ``fdisk -l`` to determine which
disk you would like to write the image to and replace ``of=/dev/vda``
with that device name.

.. code:: bash

    dd if=dcaf-abe.raw of=/dev/vda bs=64k conv=noerror,sync

**WARNING:** The dd command is unforgiving, it will overwrite a disk at
will.
