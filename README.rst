================
mainman.buildout
================

Buildout setting up Mailman 3 and postorius using Postfix as MTA.

**Note:** This buildout uses trunk checkouts of ``mailman``, ``mailman.client``
and ``postorius`` unless this packages are stable and released.


Install
=======

Install Postfix::

    apt-get install postfix

Checkout buildout::

    cd /opt
    git clone git://github.com/bluedynamics/mailman.buildout.git

Run buildout::

    cd mailman.buildout
    python bootstrap.py -c anon.cfg
    ./bin/buildout -c anon.cfg

Initialize DB::

    ./bin/django syncdb


Configure Mailman
=================

Copy ``/path/to/install/devsrc/mailman/src/mailman/config/mailman.cfg`` to
``/etc/mailman.cfg`` and adjust the following lines::

    [mta]
    incoming: mailman.mta.postfix.LMTP
    outgoing: mailman.mta.deliver.deliver
    lmtp_host: localhost
    lmtp_port: 8024
    smtp_host: localhost
    smtp_port: 25


Configure Postfix
=================

Edit ``/etc/postfix/main.cf``.

Add full qualified domain name of list(s) to ``mydestination``::

     mydestination = lists.mydomain.com mydomain.com

Add this configuration properties::

    recipient_delimiter = +
    unknown_local_recipient_reject_code = 550
    owner_request_special = no

Define transport maps::

    transport_maps =
        hash:/opt/mailman.buildout/var/data/postfix_lmtp

Define recipient maps::

    local_recipient_maps =
        hash:/opt/mailman.buildout/var/data/postfix_lmtp

Restart postfix::

    /etc/init.d/postfix/restart


Run Services
============

Start mailman::

    ./bin/mailman start

Run postorius::

    ./bin/django runserver


Notes
=====

- Mailman Domain URL needs to start with protocol schema, e.g. http://
  (URL host field in Web UI)