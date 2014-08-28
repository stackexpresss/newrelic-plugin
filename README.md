NewRelic Plugin Agent
=====================


Base Requirements
-----------------
The agent requires Python 2.6 or 2.7 and ``pip`` for installation. Individual plugin backends may require additional libraries and are detailed below.

Configuration File Note
-----------------------
The configuration file uses YAML as its format.

Installation Instructions
-------------------------
1. Install via ``pip``*:

::

    $ pip install newrelic-plugin-agent

* See ``pip`` installation instructions at http://www.pip-installer.org/en/latest/installing.html

2. Copy the configuration file example from ``/opt/newrelic-plugin-agent/newrelic-plugin-agent.cfg`` to ``/etc/newrelic/newrelic-plugin-agent.cfg`` and edit the configuration in that file.

3. Make a ``/var/log/newrelic`` directory and make sure it is writable by the user specified in the configuration file

4. Make a ``/var/run/newrelic`` directory and make sure it is writable by the user specified in the configuration file

5. Run the app:

::

    $ newrelic-plugin-agent -c PATH-TO-CONF-FILE [-f]

Where ``-f`` is to run it in the foreground instead of as a daemon.



Installing Additional Requirements
----------------------------------

For the PostgreSQL plugin you must install the ``psycopg2`` library. To easily do
this, make sure you have the latest version of ``pip`` installed (http://www.pip-installer.org/). This should be done after installing the agent itself:

::

    $ pip install newrelic-plugin-agent[postgresql]

If this does not work for you, make sure you are running a recent copy of ``pip`` (>= 1.3).

PostgreSQL Installation Notes
-----------------------------
By default, the specified user must be superuser to get PostgreSQL
directory listings. To skip those checks that require superuser
permissions, use the ``superuser: False`` setting in the configuration
file.

Several of the checks take O(N) time where N is the number of relations
in the database. If you need to use this on a database with a very large
number of relations, you can skip these, using ``relation_stats: False``.

E.g.:

::

    postgresql:
      host: localhost
      port: 5432
      user: newrelic
      dbname: postgres
      password: newrelic
      superuser: False
      relation_stats: False

Redis Installation Notes
------------------------
For Redis daemons that are password protected, add the password configuration value, otherwise omit it. The Redis configuration section allows for multiple redis servers. The syntax to poll multiple servers is in the example below.

The Redis plugin can communicate either over UNIX domain sockets using the path configuration variable or TCP/IP using the host and port variables. Do not include both.


Configuration Example
---------------------

::

    %YAML 1.2
    ---
    Application:
      license_key: REPLACE_WITH_REAL_KEY
      poll_interval: 60
      #newrelic_api_timeout: 10
      #proxy: http://localhost:8080

      postgresql:
        - host: localhost
          port: 5432
          user: postgres
          dbname: postgres
          superuser: True

      redis:
        - name: localhost
          host: localhost
          port: 6379
          db_count: 16
          password: foobar
          #path: /var/run/redis/redis.sock
        - name: localhost
          host: localhost
          port: 6380
          db_count: 16
          password: foobar
          #path: /var/run/redis/redis.sock

    Logging:
      formatters:
        verbose:
          format: '%(levelname) -10s %(asctime)s %(process)-6d %(processName) -15s %(threadName)-10s %(name) -25s %(funcName) -25s L%(lineno)-6d: %(message)s'
      handlers:
        file:
          class : logging.handlers.RotatingFileHandler
          formatter: verbose
          filename: /var/log/newrelic/newrelic-plugin-agent.log
          maxBytes: 10485760
          backupCount: 3
      loggers:
        newrelic-plugin-agent:
          level: INFO
          propagate: True
          handlers: [console, file]
        requests:
          level: ERROR
          propagate: True
          handlers: [console, file]
