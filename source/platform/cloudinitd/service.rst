===================================
Explanation of cloudinit.d services
===================================

Animated example
================

.. raw:: html

    <object width="425" height="354" id="player">
    <param name="movie" value="http://www.authorstream.com/player.swf?p=971906_634389035994692500&pt=3" />
    <param name="allowfullscreen" value="true" />
    <param name="allowScriptAccess" value="always"/>
    <embed src="http://www.authorstream.com/player.swf?p=971906_634389035994692500&pt=3" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="425" height="354"></embed>
    </object>

`link <http://www.authorstream.com/Presentation/buzztroll-971906-cloudinitd-simple/>`_

Introduction
============

The primary building block of cloudinit.d is the service.
For readers familiar with init.d, the cloudinit.d service is
akin to the *program* in init.d.  On this page we will
describe the cloudinit.d service construct and how cloudinit.d
starts and configures its services.

Often times a service is composed of many processes running in a single
VM.  This is not strictly true.  In fact, a service can be created
entirely without a VM, and further many cloudinit.d service's can all
run on the same machine.  However, in the classic case a VM is started
specifically to accommodate the needs of a single server.  That VM is
launched by cloudinit.d, a set of user defined configuration scripts are
then copied into that VM and run to contextualize it.  The healthy of that
service is then monitored by another service program.

The presentation at the beginning of this page illustrates this.

There are three programs that are associated with a service:

* ``bootpgm`` This program runs once to setup the service.  Often
  times it will download and install software (apt-get/yum) and then
  configure that software for use considering the values and locations
  of other services in the boot.
* ``readypgm`` This program is run to check the status and health
  of the service.  It can be, and typically is, run many times.  As an
  example, if the service's goal was to serve http, the readypgm would
  connect to ``localhost:80``, download a known web page and check its content.
  If all is well the readypgm returns 0 and the service is reported as
  working.  If not, the services is marked as *down* and the
  cloud application is in need of a repair.
* ``terminatepgm`` The terminate program is run when a service
  is shutdown.  It is there to nicely cleanup resources associated
  with the service.

All of these programs are user defined and written as part of the
process of creating a cloudinit.d service.  None of the programs
are mandatory.  If you can create a service that does not need one
of them, by all means ignore the unneeded ones. However we have
found that they tend to be needed.

As an example let us say that we want to create a service that runs
an apache2 web server.  We find a base ubuntu VM image on our cloud
and we associate it with our service.  Now we must create the pgms
needed to turn this VM into a cloudinit.d service.  Some possible
programs are shown below:

.. note:: These programs can be written in any scripting language available on the
    VM on which they will be run.

``bootpgm`` ::

    #!/bin/bash

    sudo apt-get install apache2
    exit $?

``readypgm`` ::

    #!/bin/bash

    wget http://localhost
    exit $?

``terminatepgm`` ::

    #!/bin/bash
    
    /etc/init.d/apache2 stop
    exit $?

All of the above programs are ``scp``'ed to the hosting machine and then
run via ssh.  Thus the user must have ssh credentials that will work on
the associated cloud and VM, and port 22 must be open to that VM.  This
is a common scenario for infrastructure clouds.

A bootlevel is a set of services that can all be started at the same time.
In other words, a set of services that have no dependencies on each other.
These services are launched at the same time.  The boot level is considered
complete when all of the services' bootpgms successfully complete.


Cloud settings
==============

All of the information about where (what cloud) a particular service is to
be run on is set at the service level.  Thus a cloudinit.d application
can run across many clouds.  It is entirely possible (and encouraged) to
have an application launched with cloudinit.d such that some services are
in a private cloud and others are in a public cloud.  Right now cloudinit.d
works on EC2, Nimbus, and Eucalyptus.  It has a very modular interface that
will allow us to quickly add other infrastructure cloud types so we expect
that list to grow quickly.


Dependencies
============

The goal of cloudinit.d is to orchestrate many services to work in concert
together to form a single application.  In order for this to be possible
the services need a way to discover information about each other.  For example,
if we are making a web server backed by a database we would make the database
one service and the webserver another (as is the case in our 
:doc:`wordpress example <wordpress>`.
Because the web server is dependent upon the database, We would
put the database at bootlevel 1 and the web server at boot level 2.

However, just having the web server wait for the database to be ready is
not enough.  The web server must know the IP address and the port number
of the data base in order to connect to it.  Further they likely need some
sort of shared secret for making a secure connection.  Cloudinit.d handles
the exchange of this, and similar types of dependency information.  Any
service is allowed to lookup another service (that is at a lower boot level)
and request an attribute from it.  There is a small set of statically
defined attributes that a service has (ex: hostname, IaaS instance id, etc)
and the service can further defined its own setup attributes.

This secure exchange of service defined attributes is what makes
cloudinit.d a powerful tool.

