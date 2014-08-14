=====================
 Preflight Checklist
=====================

.. versionadded:: 0.60

Thank you for trying Ceph! We recommend setting up a ``ceph-deploy`` admin :term:`node`
and a 3-node :term:`Ceph Storage Cluster` to explore the basics of Ceph. This
**Preflight Checklist** will help you prepare a ``ceph-deploy`` admin node and
three Ceph Nodes (or virtual machines) that will host your Ceph Storage Cluster.
Before proceeding any further, see `OS Recommendations`_ to verify that you have
a supported distribution and version of Linux.

In the descriptions below, :term:`Node` refers to a single machine.

.. include:: quick-common.rst



Ceph Deploy Setup
=================

Add Ceph repositories to the ``ceph-deploy`` admin node. Then, install
``ceph-deploy``.

Advanced Package Tool (APT)
---------------------------

For Debian and Ubuntu distributions, perform the following steps:

#. Add the release key::

	wget -q -O- 'https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc' | sudo apt-key add -

#. Add the Ceph packages to your repository. Replace ``{ceph-stable-release}``
   with a stable Ceph release (e.g., ``cuttlefish``, ``dumpling``,
   ``emperor``, ``firefly``, etc.).
   For example::

	echo deb http://ceph.com/debian-{ceph-stable-release}/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list

#. Update your repository and install ``ceph-deploy``::

	sudo apt-get update && sudo apt-get install ceph-deploy

.. note:: You can also use the EU mirror eu.ceph.com for downloading your packages.
   Simply replace ``http://ceph.com/`` by ``http://eu.ceph.com/``


Red Hat Package Manager (RPM)
-----------------------------

For Red Hat(rhel6), CentOS (el6), and Fedora 17-19 (f17-f19) perform the
following steps:

#. Add the package to your repository. Open a text editor and create a
   Yellowdog Updater, Modified (YUM) entry. Use the file path
   ``/etc/yum.repos.d/ceph.repo``. For example::

	sudo vim /etc/yum.repos.d/ceph.repo

   Paste the following example code. Replace ``{ceph-release}`` with
   the recent major release of Ceph (e.g., ``firefly``). Replace ``{distro}``
   with your Linux distribution (e.g., ``el6`` for CentOS 6, ``rhel6`` for
   Red Hat 6, ``fc18`` or ``fc19`` for Fedora 18 or Fedora 19, and ``sles11``
   for SLES 11). Finally, save the contents to the
   ``/etc/yum.repos.d/ceph.repo`` file. ::

	[ceph-noarch]
	name=Ceph noarch packages
	baseurl=http://ceph.com/rpm-{ceph-release}/{distro}/noarch
	enabled=1
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc


#. Update your repository and install ``ceph-deploy``::

	sudo yum update && sudo yum install ceph-deploy


.. note:: Some distributions (e.g., RHEL) require you to comment out
   ``Defaults requiretty`` in the ``/etc/sudoers`` file for ``ceph-deploy`` to
   work properly. If editing, ``/etc/sudoers``, ensure that you use
   ``sudo visudo`` rather than a text editor.

.. note:: You can also use the EU mirror eu.ceph.com for downloading your packages.
   Simply replace ``http://ceph.com/`` by ``http://eu.ceph.com/``


Ceph Node Setup
===============
.. important:: Do not call ``ceph-deploy`` with ``sudo`` or run it as ``root``
   if you are logged in as a different user, because it will not issue ``sudo``
   commands needed on the remote host.

If you are using ``ceph-deploy`` version 1.1.3 and beyond, ``ceph-deploy``
will attempt to create the SSH key and copy it to the initial monitor nodes
automatically when you create the new cluster.

Start from your ``ceph-deploy`` admin node. The admin node needs to
have password-less SSH access to each Ceph Node listed below.

For example, for three monitor nodes (with hostnames ``node1``, ``node2``, and
``node3``) it would look like::

	ceph-deploy new node1 node2 node3

This command will create the following files in the current directory
of the admin node:

- ``ceph.conf``
- ``ceph.log``
- ``ceph.mon.keyring``

For other Ceph Nodes in this example
(and for initial monitors prior to ``ceph-deploy`` v1.1.3)
perform the following steps:

#. Create a user on each Ceph Node. ::

	ssh user@ceph-server
	sudo useradd -d /home/ceph -m ceph
	sudo passwd ceph

#. Add ``sudo`` privileges for the user on each Ceph Node. ::

	echo "ceph ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ceph
	sudo chmod 0440 /etc/sudoers.d/ceph


#. Install an SSH server (if necessary) on each Ceph Node::

	sudo apt-get install openssh-server

   or::

	sudo yum install openssh-server


#. Configure your ``ceph-deploy`` admin node with password-less SSH access to
   each Ceph Node. When configuring SSH access, do not use ``sudo`` or the
   ``root`` user. Leave the passphrase empty::

	ssh-keygen
	Generating public/private key pair.
	Enter file in which to save the key (/ceph-client/.ssh/id_rsa):
	Enter passphrase (empty for no passphrase):
	Enter same passphrase again:
	Your identification has been saved in /ceph-client/.ssh/id_rsa.
	Your public key has been saved in /ceph-client/.ssh/id_rsa.pub.

#. Copy the key to each Ceph Node. ::

	ssh-copy-id ceph@node1
	ssh-copy-id ceph@node2
	ssh-copy-id ceph@node3


#. Modify the ``~/.ssh/config`` file of your ``ceph-deploy`` admin node so that
   it logs in to Ceph Nodes as the user you created (e.g., ``ceph``). ::

	Host node1
	   Hostname node1
	   User ceph
	Host node2
	   Hostname node2
	   User ceph
	Host node3
	   Hostname node3
	   User ceph


#. Ensure connectivity using ``ping`` with short hostnames (``hostname -s``).
   Address hostname resolution issues and firewall issues as necessary.
   **Note:** Hostnames should resolve to a network IP address, not to the
   loopback IP address (e.g., hostnames should resolve to an IP address other
   than ``127.0.0.1``).

.. note:: If you use your admin node as one of the Ceph Nodes, you must perform
   these steps on the admin node too.



Summary
=======

This completes the Quick Start Preflight. Proceed to the `Storage Cluster
Quick Start`_.

.. _Storage Cluster Quick Start: ../quick-ceph-deploy
.. _OS Recommendations: ../os-recommendations
