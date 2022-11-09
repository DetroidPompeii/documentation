============
External API
============

Odoo is usually extended internally via modules, but many of its features and
all of its data are also available from the outside for external analysis or
integration with various tools. Part of the :ref:`reference/orm/model` API is
easily available over XML-RPC_ and accessible from a variety of languages.

.. important::
   Starting with PHP8, the XML-RPC extension may not be available by default.
   Check out the `manual <https://www.php.net/manual/en/xmlrpc.installation.php>`_
   for the installation steps.

.. seealso::
   - :doc:`Tutorial on web services <../howtos/web_services>`

Connection
==========

Configuration
-------------

If you already have an Odoo server installed, you can just use its parameters.

.. important::

   For Odoo Online instances (<domain>.odoo.com), users are created without a
   *local* password (as a person you are logged in via the Odoo Online
   authentication system, not by the instance itself). To use XML-RPC on Odoo
   Online instances, you will need to set a password on the user account you
   want to use:

   * Log in your instance with an administrator account.
   * Go to :menuselection:`Settings --> Users & Companies --> Users`.
   * Click on the user you want to use for XML-RPC access.
   * Click on :guilabel:`Action` and select :guilabel:`Change Password`.
   * Set a :guilabel:`New Password` value then click :guilabel:`Change Password`.

   The *server url* is the instance's domain (e.g.
   *https://mycompany.odoo.com*), the *database name* is the name of the
   instance (e.g. *mycompany*). The *username* is the configured user's login
   as shown by the *Change Password* screen.

.. tabs::

   .. code-tab:: python

      scheme = 'https'
      domain = <insert server URL>  # 'mycompany.odoo.com'
      database = <insert database name>  # 'mycompany'
      username = 'admin'
      password = <insert password for your admin user (default: admin)>

   .. code-tab:: ruby

      scheme = 'https'
      domain = <insert server URL>  # "mycompany.odoo.com"
      database = <insert database name>  # "mycompany"
      username = "admin"
      password = <insert password for your admin user (default: admin)>

   .. code-tab:: php

      $scheme = 'https'
      $domain = <insert server URL>;  // "mycompany.odoo.com"
      $database = <insert database name>;  // "mycompany"
      $username = "admin";
      $password = <insert password for your admin user (default: admin)>;

   .. code-tab:: java

      final String scheme = "https"
            domain = <insert server URL>,  // "mycompany.odoo.com"
            database = <insert database name>,  // "mycompany"
            username = "admin",
            password = <insert password for your admin user (default: admin)>;

API Keys
~~~~~~~~

.. versionadded:: 14.0

Odoo has support for **api keys** and (depending on modules or settings) may
**require** these keys to perform webservice operations.

The way to use API Keys in your scripts is to simply replace your **password**
by the key. The login remains in-use. You should store the API Key as carefully
as the password as they essentially provide the same access to your user
account (although they can not be used to log-in via the interface).

In order to add a key to your account, simply go to your
:guilabel:`Preferences` (or :guilabel:`My Profile`):

.. image:: external_api/preferences.png
   :align: center

then open the :guilabel:`Account Security` tab, and click
:guilabel:`New API Key`:

.. image:: external_api/account-security.png
   :align: center

Input a description for the key, **this description should be as clear and
complete as possible**: it is the only way you will have to identify your keys
later and know whether you should remove them or keep them around.

Click :guilabel:`Generate Key`, then copy the key provided. **Store this key
carefully**: it is equivalent to your password, and just like your password
the system will not be able to retrieve or show the key again later on. If you lose
this key, you will have to create a new one (and probably delete the one you
lost).

Once you have keys configured on your account, they will appear above the
:guilabel:`New API Key` button, and you will be able to delete them:

.. image:: external_api/delete-key.png
   :align: center

**A deleted API key can not be undeleted or re-set**. You will have to generate
a new key and update all the places where you used the old one.

Test database
~~~~~~~~~~~~~

To make exploration simpler, you can also ask https://demo.odoo.com for a test
database:

.. tabs::

   .. code-tab:: python

      info = xmlrpc.client.ServerProxy('https://demo.odoo.com/start').start()
      url, database, username, password = \
         info['host'][8:], info['database'], info['user'], info['password']

   .. code-tab:: ruby

      info = XMLRPC::Client.new2('https://demo.odoo.com/start').call('start')
      url, database, username, password = \
         info['host'][8..-1], info['database'], info['user'], info['password']

   .. group-tab:: PHP

      .. code-block:: php

         $info = ripcord::client('https://demo.odoo.com/start')->start();
         list($url, $database, $username, $password) =
           [substr($info['host'], 8), $info['database'], $info['user'], $info['password']];

      .. note::
         These examples use the `Ripcord <https://code.google.com/p/ripcord/>`_
         library, which provides a simple XML-RPC API. Ripcord requires that
         `XML-RPC support be enabled
         <https://php.net/manual/en/xmlrpc.installation.php>`_ in your PHP
         installation.

         Since calls are performed over
         `HTTPS <https://en.wikipedia.org/wiki/HTTP_Secure>`_, it also requires that
         the `OpenSSL extension
         <https://php.net/manual/en/openssl.installation.php>`_ be enabled.

   .. group-tab:: Java

      .. code-block:: java

         final XmlRpcClient start =  new XmlRpcClient() {{
             setConfig(new XmlRpcClientConfigImpl() {{
                 setServerURL(new URL("https://demo.odoo.com/start"));
             }});
         }};
         final Map<String, String> info =
             (Map<String, String>) start.execute("start", emptyList());

         final String url = info.get("host").substring(8),
                 database = info.get("database"),
                 username = info.get("user"),
                 password = info.get("password");

      .. note::
         These examples use the `Apache XML-RPC library <https://ws.apache.org/xmlrpc/>`_.

         The examples do not include imports as these imports couldn't be
         pasted in the code.

Logging in
----------

Odoo requires users of the API to be authenticated before they can query most
data.

The ``RPC2`` endpoint provides meta-calls which don't require
authentication, such as the database management or fetching the server
version. To verify if the connection information is correct before trying
to authenticate, the simplest call is to ask for the server's version.

.. tabs::

   .. group-tab:: Python XML-RPC

      .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_xmlrpc2.py
         :language: python
         :dedent: 8
         :start-after: <a id=common>
         :end-before: </a>

   .. group-tab:: Python JSON-RPC

      .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_jsonrpc2.py
         :language: python
         :dedent: 8
         :start-after: <a id=common>
         :end-before: </a>

   .. group-tab:: Ruby

      .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
         :language: ruby
         :start-after: <a id=common>
         :end-before: </a>

   .. group-tab:: PHP

      .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.php
         :language: php
         :start-after: <a id=common>
         :end-before: </a>

   .. group-tab:: Java

      .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.java
         :language: java
         :dedent: 8
         :start-after: <a id=common>
         :end-before: </a>

   .. group-tab:: cURL

      .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.sh
         :language: bash
         :start-after: <a id=common>
         :end-before: </a>

Once the connection is established, you can connect again this time
providing a database and a user/password authentication pair. The
result should be the same as above.

.. tabs::

   .. group-tab:: Python XML-RPC

      .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_xmlrpc2.py
         :language: python
         :dedent: 8
         :start-after: <a id=models>
         :end-before: </a>

   .. group-tab:: Python JSON-RPC

      .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_jsonrpc2.py
         :language: python
         :dedent: 8
         :start-after: <a id=models>
         :end-before: </a>

   .. group-tab:: Ruby

      .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
         :language: ruby
         :start-after: <a id=models>
         :end-before: </a>

   .. group-tab:: PHP

      .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.php
         :language: php
         :start-after: <a id=models>
         :end-before: </a>

   .. group-tab:: Java

      .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.java
         :language: java
         :dedent: 8
         :start-after: <a id=models>
         :end-before: </a>

   .. group-tab:: cURL

      .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.sh
         :language: bash
         :start-after: <a id=models>
         :end-before: </a>

Result:

.. code-block:: json

   {
       "server_version": "13.0",
       "server_version_info": [13, 0, 0, "final", 0],
       "server_serie": "13.0",
       "protocol_version": 1,
   }

.. _api/external_api/calling_methods:

Calling methods
===============

When connected to a specific database, the procedure name is the concatenation
of the model name, ``.`` and the method name. The parameters are:

* a mandatory subject, which provides both the records and context to use for
  the call (if any) and can be one of:
  * a falsy value (in the Python sense so an empty collection, the boolean
  ``false``, a ``null``, the integer ``0``, ...)
  * an array (list) of record ids
  * a struct (mapping/dict) with the keys ids (an array/list of record ids) and
  context (call's context)
* an optional array of positional parameters
* an optional struct of keyword parameters

The result of the call is whatever the method returned, with a few conversions:

* returned recordsets are converted to arrays of ids
* iterables are converted to arrays of whatever they contain
* mappings are converted to structs
* mapping keys are converted to strings
* other objects are converted to structs of their vars

Depending on the API, it may also be possible to create or keep a proxy to a model on which you can keep calling methods.

.. tabs::

   .. group-tab:: Python XML-RPC

      .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_xmlrpc2.py
         :language: python
         :dedent: 8
         :start-after: <a id=check_access_rights>
         :end-before: </a>

   .. group-tab:: Python JSON-RPC

      .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_jsonrpc2.py
         :language: python
         :dedent: 8
         :start-after: <a id=check_access_rights>
         :end-before: </a>

   .. group-tab:: Ruby

      .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
         :language: ruby
         :start-after: <a id=check_access_rights>
         :end-before: </a>

   .. group-tab:: PHP

      .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.php
         :language: php
         :start-after: <a id=check_access_rights>
         :end-before: </a>

   .. group-tab:: Java

      .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.java
         :language: java
         :dedent: 8
         :start-after: <a id=check_access_rights>
         :end-before: </a>

   .. group-tab:: cURL

      .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.sh
         :language: bash
         :start-after: <a id=check_access_rights>
         :end-before: </a>

Result:

.. code-block:: json

   true


.. _external_api/search

List records
------------

Records can be listed and filtered via :meth:`~odoo.models.Model.search`.

:meth:`~odoo.models.Model.search` takes a mandatory
:ref:`domain <reference/orm/domains>` filter (possibly empty), and returns the
database identifiers of all records matching the filter.

.. example::

   To list customer companies, for instance:

   .. tabs::

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_xmlrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=list>
            :end-before: </a>

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_jsonrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=list>
            :end-before: </a>

      .. group-tab:: Ruby

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=list>
            :end-before: </a>

      .. group-tab:: PHP

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=list>
            :end-before: </a>

      .. group-tab:: Java

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.java
            :language: java
            :dedent: 8
            :start-after: <a id=list>
            :end-before: </a>

      .. group-tab:: cURL

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.sh
            :language: bash
            :start-after: <a id=list>
            :end-before: </a>


   Result:

   .. code-block:: json

      [7, 18, 12, 14, 17, 19, 8, 31, 26, 16, 13, 20, 30, 22, 29, 15, 23, 28, 74]

Pagination
~~~~~~~~~~

By default a search will return the ids of all records matching the
condition, which may be a huge number. ``offset`` and ``limit`` parameters are
available to only retrieve a subset of all matched records.

.. example::

   .. tabs::

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_xmlrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=pagination>
            :end-before: </a>

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_jsonrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=pagination>
            :end-before: </a>

      .. group-tab:: Ruby

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=pagination>
            :end-before: </a>

      .. group-tab:: PHP

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=pagination>
            :end-before: </a>

      .. group-tab:: Java

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.java
            :language: java
            :dedent: 8
            :start-after: <a id=pagination>
            :end-before: </a>

      .. group-tab:: cURL

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.sh
            :language: bash
            :start-after: <a id=pagination>
            :end-before: </a>

   Result:

   .. code-block:: json

      [13, 20, 30, 22, 29]

Count records
-------------

Rather than retrieve a possibly gigantic list of records and count them,
:meth:`~odoo.models.Model.search_count` can be used to retrieve
only the number of records matching the query. It takes the same
:ref:`domain <reference/orm/domains>` filter as
:meth:`~odoo.models.Model.search` and no other parameter.

.. example::

   .. tabs::

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_xmlrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=count>
            :end-before: </a>

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_jsonrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=count>
            :end-before: </a>

      .. group-tab:: Ruby

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=count>
            :end-before: </a>

      .. group-tab:: PHP

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=count>
            :end-before: </a>

      .. group-tab:: Java

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.java
            :language: java
            :dedent: 8
            :start-after: <a id=count>
            :end-before: </a>

      .. group-tab:: cURL

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.sh
            :language: bash
            :start-after: <a id=count>
            :end-before: </a>

   Result:

   .. code-block:: json

      19

.. note::
   Calling ``search`` then ``search_count`` (or the other way around) may not
   yield coherent results if other users are using the server: stored data
   could have changed between the calls.

Read records
------------

Record data are accessible via the :meth:`~odoo.models.Model.search_read` and :meth:`~odoo.models.Model.read` methods. ``search_read`` takes a domain whereas ``read`` takes a list of record ids (as returned by :meth:`~odoo.models.Model.search`), and optionally a list of fields to fetch. It fetches all the fields the current user can read by default. You can call the method with a non-empty ``fields`` argument to only fetch some fields.

.. example::

   This first exemple shows you how to use :meth:`~odoo.models.Model.search_read`.

   .. tabs::

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_xmlrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=search_read>
            :end-before: </a>

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_jsonrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=search_read>
            :end-before: </a>

      .. group-tab:: Ruby

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=search_read>
            :end-before: </a>

      .. group-tab:: PHP

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=search_read>
            :end-before: </a>

      .. group-tab:: Java

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.java
            :language: java
            :dedent: 8
            :start-after: <a id=search_read>
            :end-before: </a>

      .. group-tab:: cURL

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.sh
            :language: bash
            :start-after: <a id=search_read>
            :end-before: </a>

   This second example shows how to use :meth:`~odoo.models.Model.read` using
   the first record we fetched in the :ref:`external_api/search` section.

   .. tabs::

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_xmlrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=read>
            :end-before: </a>

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_jsonrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=read>
            :end-before: </a>

      .. group-tab:: Ruby

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=read>
            :end-before: </a>

      .. group-tab:: PHP

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=read>
            :end-before: </a>

      .. group-tab:: Java

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.java
            :language: java
            :dedent: 8
            :start-after: <a id=read>
            :end-before: </a>

   Result:

   .. code-block:: json

      [{"comment": false, "country_id": [21, "Belgium"], "id": 7, "name": "Agrolait"}]

.. note::
   Even if the ``id`` field is not requested, it is always returned.

List record fields
------------------

:meth:`~odoo.models.Model.fields_get` can be used to inspect
a model's fields and check which ones seem to be of interest.

Because it returns a large amount of meta-information (it is also used by client
programs) it should be filtered before printing, the most interesting items
for a human user are ``string`` (the field's label), ``help`` (a help text if
available) and ``type`` (to know which values to expect, or to send when
updating a record).

.. example::

   .. tabs::

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_xmlrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=fields_get>
            :end-before: </a>

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_jsonrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=fields_get>
            :end-before: </a>

      .. group-tab:: Ruby

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=fields_get>
            :end-before: </a>

      .. group-tab:: PHP

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=fields_get>
            :end-before: </a>

      .. group-tab:: Java

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.java
            :language: java
            :dedent: 8
            :start-after: <a id=fields_get>
            :end-before: </a>

      .. group-tab:: cURL

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.sh
            :language: bash
            :start-after: <a id=fields_get>
            :end-before: </a>

   Result:

   .. code-block:: json

      {
          "ean13": {
              "type": "char",
              "help": "BarCode",
              "string": "EAN13"
          },
          "property_account_position_id": {
              "type": "many2one",
              "help": "The fiscal position will determine taxes and accounts used for the partner.",
              "string": "Fiscal Position"
          },
          "signup_valid": {
              "type": "boolean",
              "help": "",
              "string": "Signup Token is Valid"
          },
          "date_localization": {
              "type": "date",
              "help": "",
              "string": "Geo Localization Date"
          },
          "ref_company_ids": {
              "type": "one2many",
              "help": "",
              "string": "Companies that refers to partner"
          },
          "sale_order_count": {
              "type": "integer",
              "help": "",
              "string": "# of Sales Order"
          },
          "purchase_order_count": {
              "type": "integer",
              "help": "",
              "string": "# of Purchase Order"
          },
      }

.. _external_api/create

Create records
--------------

Records of a model are created using :meth:`~odoo.models.Model.create`. The
method creates a single record and returns its database identifier.

:meth:`~odoo.models.Model.create` takes a mapping of fields to values, used
to initialize the record. For any field which has a default value and is not
set through the mapping argument, the default value will be used.

.. example::

   .. tabs::

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_xmlrpc2.py
            :language: python
            :dedent: 12
            :start-after: <a id=create>
            :end-before: </a>

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_jsonrpc2.py
            :language: python
            :dedent: 12
            :start-after: <a id=create>
            :end-before: </a>

      .. group-tab:: Ruby

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=create>
            :end-before: </a>

      .. group-tab:: PHP

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=create>
            :end-before: </a>

      .. group-tab:: Java

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.java
            :language: java
            :dedent: 8
            :start-after: <a id=create>
            :end-before: </a>

      .. group-tab:: cURL

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.sh
            :language: bash
            :start-after: <a id=create>
            :end-before: </a>

   Result:

   .. code-block:: json

      78

.. warning::
   While most value types are what would expect (integer for
   :class:`~odoo.fields.Integer`, string for :class:`~odoo.fields.Char`
   or :class:`~odoo.fields.Text`),

   - :class:`~odoo.fields.Date`, :class:`~odoo.fields.Datetime` and
     :class:`~odoo.fields.Binary` fields use string values
   - :class:`~odoo.fields.One2many` and :class:`~odoo.fields.Many2many`
     use a special command protocol detailed in :meth:`the documentation to
     the write method <odoo.models.Model.write>`.

Update records
--------------

Records can be updated using :meth:`~odoo.models.Model.write`. It takes
a list of records to update and a mapping of updated fields to values similar
to :meth:`~odoo.models.Model.create`.

Multiple records can be updated simultaneously, but they will all get the same
values for the fields being set. It is not possible to perform
"computed" updates (where the value being set depends on an existing value of
a record).

.. example::

   Because we need records on which to write, we are going to use the record
   we created in the :ref:`external_api/create` section.

   .. tabs::

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_xmlrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=write>
            :end-before: </a>

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_jsonrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=write>
            :end-before: </a>

      .. group-tab:: Ruby

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=write>
            :end-before: </a>

      .. group-tab:: PHP

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=write>
            :end-before: </a>

      .. group-tab:: Java

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.java
            :language: java
            :dedent: 8
            :start-after: <a id=write>
            :end-before: </a>

      .. group-tab:: cURL

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.sh
            :language: bash
            :start-after: <a id=write>
            :end-before: </a>

   Result:

   .. code-block:: json

      [[78, "Newer partner"]]

Delete records
--------------

Records can be deleted in bulk by providing their ids to
:meth:`~odoo.models.Model.unlink`.

.. example::

   Because we need records on which to write, we are going to use the record
   we created in the :ref:`external_api/create` section.

   .. tabs::

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_xmlrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=unlink>
            :end-before: </a>

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_jsonrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=unlink>
            :end-before: </a>

      .. group-tab:: Ruby

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=unlink>
            :end-before: </a>

      .. group-tab:: PHP

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=unlink>
            :end-before: </a>

      .. group-tab:: Java

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.java
            :language: java
            :dedent: 8
            :start-after: <a id=unlink>
            :end-before: </a>

      .. group-tab:: cURL

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.sh
            :language: bash
            :start-after: <a id=unlink>
            :end-before: </a>

   Result:

   .. code-block:: json

      []

Inspection and introspection
----------------------------

While we previously used :meth:`~odoo.models.Model.fields_get` to query a
model and have been using an arbitrary model from the start, Odoo stores
most model metadata inside a few meta-models which allow both querying the
system and altering models and fields (with some limitations) on the fly over
RPC.

.. _external_api/ir.model
.. _reference/webservice/inspection/models:

``ir.model``
~~~~~~~~~~~~

Provides information about Odoo models via its various fields.

``name``
    a human-readable description of the model
``model``
    the name of each model in the system
``state``
    whether the model was generated in Python code (``base``) or by creating
    an ``ir.model`` record (``manual``)
``field_id``
    list of the model's fields through a :class:`~odoo.fields.One2many` to
    :ref:`reference/webservice/inspection/fields`
``view_ids``
    :class:`~odoo.fields.One2many` to the :ref:`reference/views` defined
    for the model
``access_ids``
    :class:`~odoo.fields.One2many` relation to the
    :ref:`reference/security/acl` set on the model

``ir.model`` can be used to

- Query the system for installed models (as a precondition to operations
  on the model or to explore the system's content).
- Get information about a specific model (generally by listing the fields
  associated with it).
- Create new models dynamically over RPC.

.. important::
   * Custom model names must start with ``x_``.
   * The ``state`` must be provided and set to ``manual``, otherwise the model will
     not be loaded.
   * It is not possible to add new *methods* to a custom model, only fields.

.. example::

   A custom model will initially contain only the "built-in" fields available
   on all models:

   .. tabs::

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_xmlrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=ir.model>
            :end-before: </a>

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_jsonrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=ir.model>
            :end-before: </a>

      .. group-tab:: Ruby

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=ir.model>
            :end-before: </a>

      .. group-tab:: PHP

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=ir.model>
            :end-before: </a>

      .. group-tab:: Java

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.java
            :language: java
            :dedent: 8
            :start-after: <a id=ir.model>
            :end-before: </a>

      .. group-tab:: cURL

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.sh
            :language: bash
            :start-after: <a id=ir.model>
            :end-before: </a>

   Result:

   .. code-block:: json

      {
          "create_uid": {
              "type": "many2one",
              "string": "Created by"
          },
          "create_date": {
              "type": "datetime",
              "string": "Created on"
          },
          "__last_update": {
              "type": "datetime",
              "string": "Last Modified on"
          },
          "write_uid": {
              "type": "many2one",
              "string": "Last Updated by"
          },
          "write_date": {
              "type": "datetime",
              "string": "Last Updated on"
          },
          "display_name": {
              "type": "char",
              "string": "Display Name"
          },
          "id": {
              "type": "integer",
              "string": "Id"
          }
      }

.. _reference/webservice/inspection/fields:

``ir.model.fields``
~~~~~~~~~~~~~~~~~~~

Provides information about the fields of Odoo models and allows adding
custom fields without using Python code.

``model_id``
    :class:`~odoo.fields.Many2one` to
    :ref:`reference/webservice/inspection/models` to which the field belongs
``name``
    the field's technical name (used in ``read`` or ``write``)
``field_description``
    the field's user-readable label (e.g. ``string`` in ``fields_get``)
``ttype``
    the :ref:`type <reference/orm/fields>` of field to create
``state``
    whether the field was created via Python code (``base``) or via
    ``ir.model.fields`` (``manual``)
``required``, ``readonly``, ``translate``
    enables the corresponding flag on the field
``groups``
    :ref:`field-level access control <reference/security/fields>`, a
    :class:`~odoo.fields.Many2many` to ``res.groups``
``selection``, ``size``, ``on_delete``, ``relation``, ``relation_field``, ``domain``
    type-specific properties and customizations, see :ref:`the fields
    documentation <reference/orm/fields>` for details

.. important::
   - Like custom models, only new fields created with ``state="manual"`` are activated as actual
     fields on the model.
   - Computed fields can not be added via ``ir.model.fields``, some field meta-information
     (defaults, onchange) can not be set either.

.. example::

   Because we need a model on which to add a custom field, we are going to use
   the model we created in the :ref:`external_api/ir.model` section.

   .. tabs::

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_xmlrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=ir.model.fields>
            :end-before: </a>

      .. group-tab:: Python XML-RPC

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_jsonrpc2.py
            :language: python
            :dedent: 8
            :start-after: <a id=ir.model.fields>
            :end-before: </a>

      .. group-tab:: Ruby

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=ir.model.fields>
            :end-before: </a>

      .. group-tab:: PHP

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.rb
            :language: ruby
            :start-after: <a id=ir.model.fields>
            :end-before: </a>

      .. group-tab:: Java

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.java
            :language: java
            :dedent: 8
            :start-after: <a id=ir.model.fields>
            :end-before: </a>

      .. group-tab:: cURL

         .. literalinclude:: {ODOO_RELPATH}/odoo/addons/rpc2/tests/test_rpc2.sh
            :language: bash
            :start-after: <a id=ir.model.fields>
            :end-before: </a>

   Result:

   .. code-block:: json

      [
          {
              "create_uid": [1, "Administrator"],
              "x_name": "test record",
              "__last_update": "2014-11-12 16:32:13",
              "write_uid": [1, "Administrator"],
              "write_date": "2014-11-12 16:32:13",
              "create_date": "2014-11-12 16:32:13",
              "id": 1,
              "display_name": "test record"
          }
      ]

.. _PostgreSQL: https://www.postgresql.org
.. _XML-RPC: https://en.wikipedia.org/wiki/XML-RPC
.. _base64: https://en.wikipedia.org/wiki/Base64
