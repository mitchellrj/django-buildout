A Reusable Django Buildout
==========================

Intro
=====

For deploying django projects with apache and mod_wsgi in daemon mode.

Your project can be either specified as an egg to be retrieved from a python
package repository such as pypi or chishop, or can exist within the same
directory - though it must have a setup.py containing each of its dependencies.

Two apache configurations will be generated - named apache and apache-protected
where the protected configuration is exactly the same as apache but requires
basic http authentication (for details on how to configure a username/password
see below)

Usage
=====

Do not check out this file. If you want to be sure what you're referencing on
raw.github.com, point your extends directive at a particular sha rather than at
the ``master`` ref.

First, create your own ``buildout.cfg`` in your own checkout. The ``buildout.cfg``
file should contain the following::

    [buildout]
    extends = https://raw.github.com/alex2/django-buildout/master/deploy.cfg

.. note:: A new, unique secret key for your project instance can be generated
    by running the following command in your shell::

        python -c 'import random; print "".join([random.choice("abcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*(-_=+)") for i in range(50)])'

Below that, you must provide the following information::

    [project]
    name = projectname
    sitename = yoursite.example.com
    secret_key = xxxxxxxxx_see_instructions_above_xxxxxxxxxxxxxxxxx
    admin_name = Your Name
    admin_email = you@example.com

Finally, provide any or all of the following optional settings within the same
``[project]`` stanza:

Apache
------

interface
    Default value is ``*`` but should ideally be replaced with a specific ip
    address

Served Media and Static Files
-----------------------------

static-directory
    By default is ``${buildout:directory}/var/static``, and is the directory
    which will be served by apache. Collectstatic should put the files in this
    directory.

media
    Any further places to serve media from. ``${:static-directory}`` is default
    (see above) but using ``media +=`` you can append more values, such as
    upload storage directories, for example by adding::

        media += ${buildout:directory}/var/media

Python Packages
---------------

egg
    The name of your project package. Defaults to ``project:name`` but if your
    project is called "django-thingy" and your project name is "thingy", you
    should specify "django-thingy" for this value, and "thingy" as the project
    name.

eggs
    By default just contains ``project:egg``, but can be appended to with, e.g.
    ``project:eggs += django-treebeard``, if the package's setup.py file does
    not contain all the dependencies it should, or requires more packages for
    non-core functionality.

WSGI
----

user
    wsgi user, defaults to ``project:name``

group
    wsgi group, defaults to ``project:user`` (which in turn defaults to
    ``project:name`` (lazy evaluation is AWESOMES)

processgroup
    wsgi processgroup, defaults to ``project:name``

basicauth-username
    defaults to "secret"

basicauth-password
    defaults to "secret"

SSL
---

To test with some highly insecure intermediates, you can use the following
command (replacing example.com with your sitename)::

    openssl req -x509 -nodes -days 365 -subj '/C=UK/ST=North Yorkshire/L=Leeds/CN=example.com' -newkey rsa:1024 -out /etc/ssl/certs/example.com-chain.crt
    openssl req -x509 -nodes -days 365 -subj '/C=UK/ST=North Yorkshire/L=Leeds/CN=example.com' -newkey rsa:1024 -keyout /etc/ssl/private/example.com.key -out /etc/ssl/certs/example.com.crt

The SSL options are as follows:

ssl
    ``off`` by default, set to ``on`` and add cert/key settings to make the
    apache configurations ssl-enabled.

sslkey
    defaults to ``/etc/ssl/private/${project:sitename}.key``

sslcert
    defaults to ``/etc/ssl/certs/${project:sitename}.crt``

sslcachainfile
    defaults to ``/etc/ssl/certs/${project:sitename}-chain.crt`` so might need
    to be set to no value at all (``sslcachainfile =``) if your certificates
    were provided without any intermediates.

Django Project Configuration
----------------------------

It could be that you're using an upstream django project that you wish to
override particular settings for.

The following settings are overridden by default::

    ADMIN = (('${project:admin_name}', '${project:admin_email}'))
    MANAGERS = ADMINS
    SECRET_KEY = ${project:secret_key}

However more can be added, should you wish to specify more configuration via
your buildout.

Most likely you'll want to get database settings from your configuration
management database to your application. Already you can write a buildout.cfg
with all of your custom configuration from config management, but to do the
database settings as well::

    django-settings +=
        DATABASES['default']['USERNAME'] = 'dbuser'
        DATABASES['default']['PASSWORD'] = 'dbpass'
        DATABASES['default']['NAME'] = 'dbname'

You may also wish to configure your memcached servers the same way, injecting
particular IP addresses based on which part of your infrastructure you're
building the django project for.

Further Customisation
---------------------

This is just a typical buildout configuration, with all the main settings pulled
out into a single ``[project]`` stanza however, feel free to override any parts
of the other stanzas that you wish in your buildout.cfg.

Should you wish to make use of other recipe options, you can do. For example,
you may wish to set some environment variables in the django wsgi script to be
picked up by some part of your application, based on the `django recipe
documentation <https://pypi.python.org/pypi/isotoma.recipe.django>`_ you can add
the following lines to your local buildout.cfg::

    [django]
    environment.SOMEVARIABLE = "thatsabingo"

Then after running buildout, the generated bin/django and bin/django.wsgi scripts
will have::

    os.environ["SOMEVARIABLE"] = "thatsabingo"

in them, which in turn can be retrieved from the os.environ dictionary from
wherever you wish within your application.

Compatibility
-------------

Should be compatible with all versions of Django up to 1.5.
