Sharing StratusLab Responsibilities
===================================

StratusLab User Registration
----------------------------

Current procedure for user registration:
  - New users register through the registration service.
  - People on the StratusLab registration list receive requests.
  - If known academic address, approve the request.
  - If unknown address, send standard e-mail asking for more
    information.  If there's a (reasonable) response, approve it;
    otherwide ignore/deny it.
  - For approved requests, add user to user-announcements mailing
    list. 

Actions:
  - Determine who should be on list to approve users.
  - Give those people access to StratusLab Google Apps administrator
    account.
  - Move back to mail account at LAL for sending notification emails
    from registration service (currently, Cal's SixSq mailer account).

VD SlipStream Service
---------------------

Currently running in a virtual machine on the LAL infrastructure
started by Cal.  The data for this instance is kept on a StratusLab
persistent disk, also owned by Cal.

Actions:
  - Upgrade to standard community edition of the service.  (This
    edition should appear in the next few weeks.) 
  - Determine what physical machine to host the service on.
  - Migrate the service from the virtual machine to the identified
    virtual machine.

Jenkins
-------

Usually deployed in virtual machines by Cal.  All state data held on a
persistent disk in StratusLab (also owned by Cal).  Scripts to
recreate the slaves are available (and could reasonably be moved to
SlipStream).  Currently the slaves are not running because they were
killed to free space for the AppStat tutorial last week.

Actions:
  - Restart the Jenkins server/slaves and ensure that all of the
    StratusLab build jobs still execute correctly. 
  - Determine how to migrate the Jenkins infrastructure into
    SlipStream.  (The only real question is how to specify mounting
    the persistent disk on the server.)
  - Create generic SlipStream account to manage the SlipStream build
    infrastructure and deploy the continuous integration services
    through SlipStream.

Test Machines
-------------

There are two sets of test machines set up to do full integration
tests: onehost-5/6 (iscsi persistent disk) and onehost-2/10 (CEPH
persistent disk).

Currently onehost-5/6 are running base installations of CentOS 7.  The
necessary additional configuration is not yet done in the kickstart
file (but exists).  This configuration needs to be uncommented and
tested to be sure that the machines come up to a usable state.  After
that, the full distribution needs to be tested using the standard
Jenkins test jobs.

Note that after the installation of the base OS and necessary SSH
keys, the full installation and test of the current StratusLab
snapshot is handled by Jenkins.

The other test installation (onehost-2/10) was intended for deploying
CEPH.  This has never been done for lack of time, although I have a
script for doing so and the CEPH backend script for the pdisk server
exists.  These machines could be used for other purposes
(e.g. automated image creation) or the CEPH installation could be
tested. 

Actions:
  - Fix kickstart files for onehost-5/6 and ensure that full system
    installation works on CentOS 7.
  - Debug any problems identified with standard tests and ensure that
    the full system is working.
  - Determine what to do with onehost-2/10. 

Automated Image Creation
------------------------

I had discussed this with Grigory before leaving and he's sent me an
email about it, that I've not responded to.  In any case, it is
important to get to the point where all of the standard base images
are regenerated automatically every week or so.  This reduces the
effort required to keep the images up to date while ensuring that they
stay current with any security patches.

Actions:
  - Determine what machines to use for the image generation. 
  - Determine how to automate the image creation procedure.
  - Determine how to drive the image generation from Jenkins.
  - Test/fix ability to use image tags (rather than raw image IDs) to
    start an image.  This makes it easy for users to identify the
    latest standard image without having to know the raw image ID.

Adding Back X509 Certificate Support
------------------------------------

In the latest release, the deployment architecture was changed to use
nginx as a proxy in front of all of the StratusLab services.  This was
done so that only the standard HTTPS port (443) is used for all
services.  This makes accessing StratusLab possible even on sites that
have bizzare firewall rules for their wireless networks.

Supporting X509 with this deployment architecture requires:
  - Doing the standard EGI certificate configuration for
    nginx. (Should be done in a configurable way such that those
    people who don't want the configuration for X509 don't need this.)
  - Ensuring that nginx sends the client certificate to the StratusLab
    services via a special header.  (This should already be there, but
    hasn't been tested.)
  - Create a utility that given a client certificate will extract the
    user DN and roles/groups, assuming that the certificate is valid.
    (Should just be a simple call to the EGI VOMS library.)
  - Call that utility from the StratusLab login process for all of the
    StratusLab services.


Path to CIMI
------------

While most of the basic infrastucture code exists for moving to CIMI,
there are lots of things to do to get it into production.  I think
that the best path for moving forward is to:
  - Get actual disk creation working from the CIMI server.  This
    involves getting the "persistent disk controller" (in python)
    connected to the real storage back end.
  - Rework the disk cache mechanisms to use CIMI rather than the pdisk
    server.  This involves reworking the various OpenNebula hook
    scripts to call the CIMI server rather than pdisk.  (OpenNebula
    will eventually disappear, but keeping a fully running system is
    important to ensure that we don't break any functionality
    unintentionally.)
  - Remove the pdisk service entirely and directly call the storage
    backend interface directly.
  - Develop "libvirt controller" to launch machines directly from CIMI
    into libvirt.  This should essentially be a duplicate of the
    persistent disk controller but taking "Machine" entries from the
    database and creating VMs directly with libvirt.
  - Once the libvirt controller is in place, rework standard commands
    to use CIMI rather than the OpenNebula RPC interface.
  - Remove OpenNebula.

Client
------

There is an open question on whether to update/modify the current
StratusLab client or to start on a new foundation with a new client. 

The main argument for sticking with the current client is mainly
backward compatibility.  Many people are used to using that client and
some have scripted behavior with it.  The main problems are that it
has grown up organically over time, so the internal APIs and
configuration management are frankly a mess.  From experience,
modifying this code and maintaining it takes a significant amount of
effort.

The main arguments for starting from scratch with a new client are: 1)
the amount of code could be significantly reduced by using one of the
(now available) CLI support libraries, 2) the API based on CIMI would
be consistent across all resources further reducing the code, 3) a
"git-like" single command (with option completion) could be offered
easily.

A proof-of-concept CIMI client already exists, but this would need to
be significantly reworked to make it a production CLI.  It is possible
that the old CLI could be patched to work over the new API providing
some level of backward compatibility.  However there will be at a
certain level a semantic mismatch between the CIMI API and the
existing, homegrown StratusLab API.  The backwards compatibility won't
be perfect.

The decision on which route to take should be taken before much more
work is done on the CIMI migration.  This will affect where to put the
limited development effort on the client side.

Actions:
  - Decision on the desirable path for client development.
