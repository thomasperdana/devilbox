.. _tutorial_work_inside_the_php_container:

*************************
Work inside the container
*************************

The Devilbox allows you to completely work inside the PHP container (no matter what version),
instead of your host operating system.

This brings a lot of advantages, such as that you don't
have to install any development tool on your OS or if you are on Windows, you get a full blown
Linux environment.

Additionally, special port-bindings and forwards are in place that allows you to even
interchangably work locally or inside the container without having to alter any php config for
database and other connections.

.. seealso:: :ref:`available_tools`


**Table of Contents**

.. contents:: :local:


Enter the container
===================

Entering the computer is fairly simple. The Devilbox ships with two scripts to do that. One for
Linux and MacOS (``shell.sh``) and another one for Windows (``shell.bat``).

Entering from Linux or MacOS: ``shell.sh``
------------------------------------------

.. code-block:: bash

    # Navigate to the Devilbox directory
    host> cd /path/to/devilbox

    # Run provided script
    host> ./shell.sh

    # Now you are inside the PHP Linux container
    devilbox@php-7.0.19 in /shared/httpd $


Entering from Windows: ``shell.bat``
------------------------------------

.. code-block:: bash

    # Navigate to the Devilbox directory
    C:/> cd C:/Users/user1/devilbox

    # Run provided script
    C:/Users/user1/devilbox> shell.bat

    # Now you are inside the PHP Linux container
    devilbox@php-7.0.19 in /shared/httpd $


Inside the container
====================

``devilbox`` user
-----------------

By using the provided scripts to enter the container you will become the user ``devilbox``.
This user will have the same uid and gid as the user from your host operating system.

So no matter what files or directories you create inside the container, they will have the same
permissions and uid/gid set your host operating system. This of course also works the other way
round.

The uid and gid mappings are controlled via two ``.env`` variables called :ref:`env_new_uid` and
:ref:`env_new_gid`


.. seealso::
    If you want to find out more about synronized container permissions read up here:
    :ref:`syncronize_container_permissions`


``root`` user
-------------

Sometimes however it is also necessary to do some actions that require super user privileges.
You can always become root inside the container by either impersonating it or by using ``sudo``
to issue commands.

By default ``sudo`` is configured to be used without passwords, so you can simply do the following:

.. code-block:: bash

    # As user devilbox inside the container
    devilbox@php-7.0.19 in /shared/httpd $ sudo su -

    # You are now the root user
    root@php-7.0.19 in /shared/httpd $

You can also use ``sudo`` to run commands with root privileges without having to become root first.

.. code-block:: bash

    # As user devilbox inside the container
    devilbox@php-7.0.19 in /shared/httpd $ sudo apt update
    devilbox@php-7.0.19 in /shared/httpd $ sudo apt install nmap


Leave the container
===================

When you are inside the container and want to return to your host operating, just type ``exit``
and you are out.

.. code-block:: bash

    # As user devilbox inside the container
    devilbox@php-7.0.19 in /shared/httpd $ exit

    # You are now back on your host operating system
    host>


Host to Container mappings
==========================

This section will give you an idea that there is actually no difference from inside the container
and on your host operating system. Directory permissions, IP addresses, ports and DNS entries
are fully syncronized allowing you to switch between container and host without having to
change any settings.


File and directory Permissions
------------------------------

The username inside the container (``devilbox``) might be different from your local host operating
system username, however its actual uid and gid will match. This is to ensure file and directory
permissions are synronized inside and outside the container and no matter from which side you
create files and directories, it will always look as if they are owned by your system user.

The uid and gid mappings are controlled via two ``.env`` variables called :ref:`env_new_uid` and
:ref:`env_new_gid`


Directory mappings
------------------

One thing you should understand is the relation between the directories on your host operating
system and the corresponding directory inside the PHP container.

The location of the data directory (:ref:`env_httpd_datadir`) on your host computer is controlled
via the ``HOST_PATH_HTTPD_DATADIR`` variable inside the ``.env`` file. No matter what location you
set it to, inside the container it will always be mapped to ``/shared/httpd``.

See the following table for a few examples:

+----------+-----------------------+----------------------+
|          | Host operating system | Inside PHP container |
+==========+=======================+======================+
| Data dir | ``./www/data``        | ``/shared/httpd``    |
+----------+-----------------------+----------------------+
| Data dir | ``/home/user1/www``   | ``/shared/httpd``    |
+----------+-----------------------+----------------------+
| Data dir | ``/var/www``          | ``/shared/httpd``    |
+----------+-----------------------+----------------------+


IP address mappings
-------------------

The following table shows a mapping of IP addresses of available service from the perspective
of your host operating system and from within the PHP container.

+--------------+-----------------+------------------------------+
| Service      | IP from host os | IP from within PHP container |
+==============+=================+==============================+
| PHP          | ``127.0.0.1``   | ``127.0.0.1``                |
+--------------+-----------------+------------------------------+
| Apache/Nginx | ``127.0.0.1``   | ``127.0.0.1``                |
+--------------+-----------------+------------------------------+
| MySQL        | ``127.0.0.1``   | ``127.0.0.1``                |
+--------------+-----------------+------------------------------+
| PostgreSQL   | ``127.0.0.1``   | ``127.0.0.1``                |
+--------------+-----------------+------------------------------+
| Redis        | ``127.0.0.1``   | ``127.0.0.1``                |
+--------------+-----------------+------------------------------+
| Memcached    | ``127.0.0.1``   | ``127.0.0.1``                |
+--------------+-----------------+------------------------------+
| MongoDB      | ``127.0.0.1``   | ``127.0.0.1``                |
+--------------+-----------------+------------------------------+

As you can see, everyhing is available under ``127.0.0.1``.

The PHP container is using ``socat`` to forward the services from all other available containers
to its own ``127.0.0.1`` address.

An example to access the MySQL database from either host or within the PHP container is the same:

.. code-block:: bash

    # Access MySQL from your host operating system
    host> mysql -h 127.0.0.1

    # Access MySQL from within the PHP container
    devilbox@php-7.0.19 in /shared/httpd $ mysql -h 127.0.0.1

.. important::
    Do not use ``localhost`` to access the services, it does not map to ``127.0.0.1`` on
    all cases.

So when setting up a configuration file from your PHP project you would for example use
``127.0.0.`` as the host for your MySQL database connection:

.. code-block:: php

    <?php
    // MySQL server connection
    mysql_host = '127.0.0.1';
    mysql_port = '3306';
    mysql_user = 'someusername';
    mysql_pass = 'somepassword';
    ?>

Imagine your PHP framework ships a command line tool to run database migration. You could run
it from your host operating system or from within the PHP container. It would work from both
sides as the connection to the database is exactly the same locally or within the container.

You could also even switch between the Devilbox and a locally installed LAMP stack
and still use the same configuration.

.. warning::
    The mapping of ``127.0.0.1`` to your host operating system does not work with
    :ref:`docker_toolbox`.


Port mappings
-------------

By default, ports are also synronized between host operating system (the ports that are exposed)
and the ports within the PHP container. This is however also configurable inside the ``.env`` file.

+--------------+-------------------+--------------------------------+
| Service      | Port from host os | Port from within PHP container |
+==============+===================+================================+
| PHP          | NA                | ``9000``                       |
+--------------+-------------------+--------------------------------+
| Apache/Nginx | ``80``            | ``80``                         |
+--------------+-------------------+--------------------------------+
| MySQL        | ``3306``          | ``3306``                       |
+--------------+-------------------+--------------------------------+
| PostgreSQL   | ``5432``          | ``5432``                       |
+--------------+-------------------+--------------------------------+
| Redis        | ``6379``          | ``6379``                       |
+--------------+-------------------+--------------------------------+
| Memcached    | ``11211``         | ``11211``                      |
+--------------+-------------------+--------------------------------+
| MongoDB      | ``27017``         | ``27017``                      |
+--------------+-------------------+--------------------------------+


DNS mappings
------------

All project DNS records are also available from inside the PHP container independent of the
value of :ref:`env_tld_suffix`.

The PHP container is hooked up by default to the bundled DNS server and makes use
:ref:`global_configuration_auto_dns`.

.. seealso::
    You can achieve the same on your host operating system by explicitly enabling auto-dns.
    See also: :ref:`global_configuration_auto_dns`.


Checklist
=========

1. You know how to enter the PHP container
2. You know how to become root inside the PHP container
3. You know how to leave the container
4. You know that file and directory permissions are synronized
5. You know that ``127.0.0.1`` is available on your host and inside the PHP container
6. You know that ports are the same inside the container and on your host os
7. You know that project urls are available inside the container and on your host
8. You know about the limitations of :ref:`docker_toolbox`
