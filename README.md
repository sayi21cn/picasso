Project LaOS aka Lambdas-on-OpenStack
=====================================

Mission
-------

Provide capabilities to run software in "serverless" way.

Serverless
----------

Serverless is a new paradigm in computing that enables simplicity, 
efficiency and scalability for both developers and operators. 
It's important to distinguish the two, because the benefits differ:

Benefits for developers
-----------------------

The main benefits that most people refer to are on the developer side and they include:

* No servers to manage (serverless) -- you just upload your code and the platform deals with the infrastructure
* Super simple coding -- no more monoliths! Just simple little bits of code
* Pay by the milliseconds your code is executing -- unlike a typical application that runs 24/7, and you're paying
  24/7, functions only run when needed

Benefits for operators
----------------------

If you will be operating IronFunctions (the person who has to manage the servers behind the serverless),
then the benefits are different, but related.

* Extremely efficient use of resources
  * Unlike an app/API/microservice that consumes resources 24/7 whether they
    are in use or not, functions are time sliced across your infrastructure and only consume resources while they are
    actually doing something
* Easy to manage and scale
  * Single system for code written in any language or any technology
  * Single system to monitor
  * Scaling is the same for all functions, you don't scale each app independently
  * Scaling is simply adding more IronFunctions nodes

System requirements
-------------------

* Operating system: Linux/MacOS
* Python version: 3.5 or greater
* Database: MySQL 5.7 or greater

Quick-start guide
-----------------

Install DevStack with [IronFunctions enabled](https://github.com/iron-io/functions-devstack-plugin/blob/master/README.rst).
Pull down [Project LaOS sources](https://github.com/iron-io/project-laos).

Create Python3.5 virtualenv:

    $ virtualenv -p python3.5 .venv
    $ source .venv/bin/activate

Install dependencies:

    $ pip install -r requirements.txt -r test-requirements.txt

Install LaOS itself:

    $ pip install -e .

Install MySQL if you haven't already, and create a new database for functions.

    $ mysql -uroot -p -e "CREATE DATABASE functions"


Migrations
----------

Once all dependencies are installed it is necessary to run database migrations.
Before that please edit [alembic.ini](alembic.ini) line #32

    sqlalchemy.url = mysql+pymysql://root:root@localhost/functions

In this section please specify connection URI to your own MySQL database.
Once the file is saved, just use alembic to apply the migrations:

    $ alembic upgrade head

Starting a server
-----------------

Once it is finished you will have a console script `laos-api`:

    $ laos-api --help

    Usage: laos-api [OPTIONS]
    
      Starts an Project Laos API service
    
    Options:
      --host TEXT                    API service bind host.
      --port INTEGER                 API service bind port.
      --db-uri TEXT                  LaOS persistence storage URI.
      --keystone-endpoint TEXT       OpenStack Identity service endpoint.
      --functions-host TEXT          Functions API host
      --functions-port INTEGER       Functions API port
      --functions-api-version TEXT   Functions API version
      --functions-api-protocol TEXT  Functions API protocol
      --log-level TEXT               Logging file
      --log-file TEXT                Log file path
      --help                         Show this message and exit.

Minimum required options to start LaOS API service:

     --db-uri mysql://root:root@192.168.0.112/functions
     --keystone-endpoint http://192.168.0.112:5000/v3
     --functions-host 192.168.0.112
     --functions-port 10501
     --log-level INFO

Examining API
-------------

In [examples](examples/) folder you can find a script that examines available API endpoints, but this script relays on:

* `LAOS_API_URL` - Project LaOS API endpoint
* `OS_AUTH_URL` - OpenStack Auth URL
* `OS_PROJECT_ID` - it can be found in OpenStack Dashboard or in CLI
* `OS_USERNAME` - OpenStack project-aligned username
* `OS_PASSWORD` - OpenStack project-aligned user password
* `OS_DOMAIN` - OpenStack project domain name
* `OS_PROJECT_NAME` - OpenStack project name

Then just run script:

    OS_AUTH_URL=http://192.168.0.112:5000/v3 OS_PROJECT_ID=8fb76785313a4500ac5367eb44a31677 OS_USERNAME=admin OS_PASSWORD=root OS_DOMAIN=default OS_PROJECT_NAME=admin ./examples/hello-lambda.sh

Please note, that given values are project-specific, so they can't be reused.

API docs
--------

As part of LaOS ReST API it is possible to discover API doc using Swagger Doc.
Once server is launched you can navigate to:

    http://<laos-host>:<laos-port>/api

to see recent API docs

Testing (general information)
-----------------------------

In order to run tests you need to install `Tox`:

    $ pip install tox

Also, you will need a running MySQL instance with the database migrations applied from the previous step.
Tests are dependent on pre-created MySQL database for persistence.
Please set env var

    $ export TEST_DB_URI=mysql://<your-user>:<your-user-password>@<mysql-host>:<mysql-port>/<functions-db>

Testing: PEP8
-------------

In order to run `PEP8` style checks run following command:

    $ tox -e pep8


Testing: Functional
-------------------

In order to run `functional` tests run following command:

    $ tox -e py35-functional

Pros:

* lightweight (controllers and DB models testing)
* no OpenStack required
* no IronFunctions required

Cons:

* MySQL server required
* OpenStack authentication is not tested
* IronFunctions API stubbed with fake implementation

Testing: Integration
--------------------

Integration tests are dependent on following env variables:

* TEST_DB_URI - similar to functional tests, database endpoint
* FUNCTIONS_API_URL - IronFunctions API URL (default value - `http://localhost:8080/v1`)
* OS_AUTH_URL - OpenStack Identity endpoint
* OS_PROJECT_NAME - OpenStack user-specific project name
* OS_USERNAME - OpenStack user name
* OS_PASSWORD - OpenStack user user password

To run tests use following command:

    export TEST_DB_URI=mysql://<your-user>:<your-user-password>@<mysql-host>:<mysql-port>/<functions-db>
    export FUNCTIONS_API_URL=<functions-api-protocol>://<functions-host>:<functions-port>/<functions-api-version>
    export OS_AUTH_URL=<identity-api-protocol>://<identity-host>:<identity-port>/<identity-api-version>
    export OS_PROJECT_NAME=<project-name>
    export OS_USERNAME=<project-name>
    export OS_PASSWORD=<project-name>
    tox -epy35-integration

Testing: Coverage regression
----------------------------

In order to build quality software it is necessary to keep test coverage at its highest point.
So, as part of `Tox` testing new check was added - functional test coverage regression.
In order to run it use following command:

    $ tox -e py35-functional-regression

3rd party bugs to resolve
-------------------------

IronFunctions:

* https://github.com/iron-io/functions/issues/298
* https://github.com/iron-io/functions/issues/296
* https://github.com/iron-io/functions/issues/275
* https://github.com/iron-io/functions/issues/274

TODOs
-----

Swagger doc:

* Make swagger doc more explicit on HTTP POST/UPDATE body content
* HTTP headers requests

IronFunctions:

* Support app deletion in IronFunctions
* Support tasks listing/showing

Laos:

* Tests: integration, functional, units
* Better logging coverage

Python Functions client:

* Support logging instance passing in [function-python](https://github.com/iron-io/functions_python)
* python-laosclient (ReST API client and CLI tool)
* App writing examples


Contacts
--------

Feel free to reach us out at:

* [Slack channel](https://open-iron.herokuapp.com/)
* [Email](https://github.com/denismakogon)
