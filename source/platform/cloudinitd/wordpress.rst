=================
Wordpress example
=================


Introduction
============

This example will launch an EC2 cloud application which runs a
`wordpress <http://www.wordpress.com>`_ system.  Two virtual machines
will be launched, the first runs a `MySQL <http://www.mysql.com>`_
database, and the second runs the wordpress service.

.. warning::
    Running VMs on EC2 requires an EC2 account which will be charged.  At the
    time of this writing the rates for an m1.small instance is $0.085 per hour.
    Rates can be checked `here <http://aws.amazon.com/ec2/pricing/>`_.


Quickstart
==========

Once you have cloudinit.d installed the following commands will get this
example, boot it in EC2, and present you will a functional wordpress service. ::

    $ export CLOUDBOOT_IAAS_ACCESS_KEY=<your EC2 access key>
    $ export CLOUDBOOT_IAAS_SECRET_KEY=<your EC2 secret key>
    $ export CLOUDBOOT_IAAS_SSHKEY=<the path to your ssh key>
    $ export CLOUDBOOT_IAAS_SSHKEYNAME=<the name of your ssh key in EC2>
    $ wget http://www.nimbusproject.org/downloads/wordpress.tar.gz
    $ tar -zxf wordpress.tar.gz
    $ cloudinitd -v -v -v boot wordpress/top.conf

When this completes a full wordpress service will be ready for your use.
The output of cloudinit.d will look something like this:

.. code-block:: none

    Starting up run c80e7e2c
        Started IaaS work for mysql
        Started IaaS work for wordpress
    Starting the launch plan.
    Begin boot level 1...
        Started mysql
            SUCCESS service mysql boot
            hostname: ec2-50-17-78-14.compute-1.amazonaws.com
            instance: i-f8380797
    SUCCESS level 1
    Begin boot level 2...
    Begin boot level 2...
        Started wordpress
        SUCCESS service wordpress boot
            hostname: ec2-174-129-163-229.compute-1.amazonaws.com
            instance: i-ca3807a5
    SUCCESS level 2

Note the second hostname printed under the ``wordpress`` service
information: ``ec2-174-129-163-229.compute-1.amazonaws.com``.  Simply
point your web browser at
``http://ec2-174-129-163-229.compute-1.amazonaws.com/wordpress/wp-admin/install.php``
and you have your own personal wordpress service!

.. note:
    For this example to work you need your default security group to have
    port 22, 80 and 3306 open.


Cleanup
=======

You can terminate the entire cloud application with a single command as
well. The first thing to note is the *run name* which is printed
out in the first line of the boot output::

    Starting up run c80e7e2c

In out case it is ``c80e7e2c``.  To terminate the launch run the following command::

    $ cloudinitd -v -v -v terminate c80e7e2c
    Terminating c80e7e2c
    Begin terminate level 2...
        Started wordpress
        SUCCESS service wordpress terminate
            hostname: None
            instance: i-ca3807a5
    SUCCESS level 2
    Begin terminate level 1...
    Begin terminate level 1...
        Started mysql
        SUCCESS service mysql terminate
            hostname: None
            instance: i-f8380797
    SUCCESS level 1
    deleting the db file /home/bresnaha/.cloudinitd/cloudinitd-c80e7e2c.db

All of the resources associated with your cloud application will now
be cleaned up.




What Happened
=============

The following `presentation <http://www.mcs.anl.gov/~bresnaha/cid_wp.wmv>`_ 
shows what happened.

.. raw:: html

    <OBJECT ID="MediaPlayer"  WIDTH="720" HEIGHT="540" 
    STANDBY="Loading Windows Media Player components..." TYPE="application/x-oleobject">
    <PARAM NAME="FileName" VALUE="cid_wp.wmv">
    <PARAM name="autostart" VALUE="false">
    <PARAM name="ShowControls" VALUE="true">
    <param name="ShowStatusBar" value="false">
    <PARAM name="ShowDisplay" VALUE="false">
    <EMBED TYPE="application/x-mplayer2" SRC="http://www.mcs.anl.gov/~bresnaha/cid_wp.wmv" NAME="MediaPlayer"
    WIDTH="720" HEIGHT="540" ShowControls="1" ShowStatusBar="0" ShowDisplay="0" autostart="0"> </EMBED>
    </OBJECT>


Launch plan
===========

The details of the launch are found in the launch plan.  The first file is ``top.conf``::

    [defaults]
    iaas_key: env.CLOUDBOOT_IAAS_ACCESS_KEY
    iaas_secret: env.CLOUDBOOT_IAAS_SECRET_KEY
    localsshkeypath: env.CLOUDBOOT_IAAS_SSHKEY
    sshkeyname: env.CLOUDBOOT_IAAS_SSHKEYNAME

    [runlevels]
    level1: mysql_level.conf
    level2: wp_level.conf

Here we see above that key security information is gathered from the
environment variables (this is why we had to set the prior to launch
in the quick start).  We also see that there are two run levels.  The
first handles the MySQL server, and once that is done, The second uses
it to handle the wordpress service.

If we look at the two run level files ``mysql_level.conf`` and
``wp_level.conf`` we see that each has a section that starts with
``svc``.  What follows ``svc-`` is the name of the service to be
described.  In ``svc-wordpress`` and ``svc-mysql``
we see three similar values and three different ones.
First lets look at the similar values::

    ssh_username: ubuntu
    image: ami-ccf405a5
    allocation: m1.small

These lines tell cloudinit.d to launch the image name ``ami-ccf405a5``
(this is a standard ubuntu10.10 image) as a m1.small instance.  The
``ssh_username`` tells cloudinit.d which username can be accessed with
the previously established keys.

Those line are enough to establish two base virtual machines in the associated
cloud.  From there the next  thing to do is customize these VMs to do their
needed jobs, become a mysql server and a wordpress server.  The next three
lines of the configuration file handle this.

``bootpgm`` points to a script that is copied into the virtual machine
where it is run.  This script should download, install, and configure
the machine to do its job.  In the case of the MySQL server software
is installed with apt-get and configured.  In the wordpress case
wordpress is downloaded and installed.

Further, the hostname where
the MySQL service is running is passed to the wordpress VM so that it
can connect to it.  This is handled with the ``deps`` directive and the
``bootconf`` directive.  The files in this launch plan serve as a good
example for how this works.


Troubleshooting
===============

When a service is launched a series of log files are created under:
``~/.cloudinitd/<run name>``.  Valuable information about the progress
of a launch can be found in these directories.


