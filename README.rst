================
mailman.buildout
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

While DB is initialized, system asks for superuser creation.


Building Documentation
======================

Mailman sources include a sphinx documentation. In order to generate HTML docs,
sphinx must be installed in your python. Then you can navigate to::

    $mailman_install_dir/devsrc/mailman

and run::

    python setup.py build_sphinx

Generated docs are not located at::

    $mailman_install_dir/devsrc/mailman/build/sphinx/html

and can be viewed with any web browser.


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

Postorius binds to 127.0.0.1:8000 by default


Create Domain
=============

Open browser and navigate to portorius (127.0.0.1:8000).

Navigate to settings and klick "new domain".

Enter mail host::

    lists.example.com

Enter web host::

    lists.example.com

Mailman Domain web host needs to start with protocol schema, e.g. http://.
At time of writing this document, postorius has a validation bug in domain form
validation, where no protocol scheme can be defined. In order to fix this, use
mailman shell via command line by typing::

    ./bin/mailman shell

and finish domain configuration::

    >>> from mailman.interfaces.domain import IDomainManager
    >>> from zope.component import getUtility
    >>> manager = getUtility(IDomainManager)
    >>> domain = manager.get(u'lists.example.com')
    >>> domain.base_url = u'http://lists.example.com' 

The contact address should also be changed with::

    >>> domain.contact_address = u'postmaster@example.com'

Save changes::

    >>> commit()


Create Mailinglist
==================

Start creating mailinglist in postorius. Navigate to "lists" and click
"new list".

Enter list name, e.g. ``test``.

Choose mailhost.

Define list owner (defaults to domain contact)

Choose whether to advertise list

Enter description

Save list.

Navigate to "settings" -> "message acceptance" and set default action for
member posts to "Hold for moderator"


Creating a moderator for list
=============================

Open mailman shell with mailinglist::

    ./bin/mailman withlist test@lists.example.com

Create user which gets moderator in list::

    >>> from mailman.interfaces.usermanager import IUserManager
    >>> from zope.component import getUtility
    >>> user_manager = getUtility(IUserManager)
    >>> moderator = user_manager.create_user(u'foo@example.com', u'Foo')

Add to list as moderator::

    >>> from mailman.interfaces.member import MemberRole
    >>> address = list(moderator.addresses)[0]
    >>> m.subscribe(address, MemberRole.moderator)

Save changes::

    >>> commit()


Setting a password for a user
=============================

Open mailman shell::

    ./bin/mailman shell

Set password::

    >>> from mailman.interfaces.usermanager import IUserManager
    >>> from zope.component import getUtility
    >>> user_manager = getUtility(IUserManager)
    >>> user = user_manager.get_user(u'foo@example.com')
    >>> user.password = 'secret'

Save changes::

    >>> commit()

XXX: authenticate in postorius with this user?


Adopting templates for mailinglist
==================================

Open mailman shell with mailinglist::

    ./bin/mailman withlist test@lists.example.com

Set templates. Can be any URI supported by ``urllib2`` with the addition of
``mailman:`` URIs::

    >>> m.welcome_message_uri = u'mailman:///welcome.txt'
    >>> m.goodbye_message_uri = u'file:///path/to/file.txt'
    >>> m.header_uri = u'mailman:///$listname/$language/header-generic.txt'
    >>> m.footer_uri = u'mailman:///$listname/$language/footer-generic.txt'
    >>> commit()

XXX: Correct links in email

XXX: Template locations and assets

Note: The Web confirmation link is completly broken in postorius, remove it
from templates, only email confirmation possible right now.


Subscribe to Mailinglist
========================

Send an email to
    test-request@lists.example.com

with subject
    subscribe

and message
    subscribe

``test`` is desired mailinglist


Unsubscribe from Mailinglist
============================

Send an email to
    test-request@lists.example.com

with subject
    unsubscribe

and message
    unsubscribe

``test`` is desired mailinglist


Create postorius user
=====================

Create a postorius user permitted to moderate list::

    ./bin/django shell
    
    >>> from django.contrib.auth.models import User
    >>> user = User.objects.create_user(username='john', email='', password='x')
    >>> user.is_staff = False
    >>> user.save()

Check authentication::

    >>> from django.contrib.auth import authenticate
    >>> authenticate(username="john", password="x")
    <User: phonogen>


Configure collective.newsletter
===============================

Install ``collective.newsletter`` in your plone instance.

Install ``collective.newsletter`` from ``addons`` in control panel.

Create mailing list in ``newsletter`` addon configuration.

Give it a name, and a list email address.

Choose mailman as protocol.
