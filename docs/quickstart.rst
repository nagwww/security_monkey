=================
Quick Start Guide
=================

Docker Images
=============

Before we start, consider following the `docker instructions <https://github.com/Netflix-Skunkworks/zerotodocker/wiki/Security-Monkey>`_
. Docker helps simplify the process to get up and running.  The docker images are not currently ready for production use, but are good enough to get up and running with an instance of security_monkey.

Local `docker instructions <./docker.html>`_

Not into the docker thing? Keep reading.

Setup IAM Roles
===============

We need to create two roles for security monkey.  The first role will be an
instance profile that we will launch security monkey into.  The permissions
on this role allow the monkey to use STS to assume to other roles as well as
use SES to send email.

Creating SecurityMonkeyInstanceProfile Role
-------------------------------------------

Create a new role and name it "SecurityMonkeyInstanceProfile":

.. image:: images/resized_name_securitymonkeyinstanceprofile_role.png

Select "Amazon EC2" under "AWS Service Roles".

.. image:: images/resized_create_role.png

Select "Custom Policy":

.. image:: images/resized_role_policy.png

Paste in this JSON with the name "SecurityMonkeyLaunchPerms":

.. code-block:: json

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "ses:SendEmail"
          ],
          "Resource": "*"
        },
        {
          "Effect": "Allow",
          "Action": "sts:AssumeRole",
          "Resource": "arn:aws:iam::*:role/SecurityMonkey"
        }
      ]
    }

Review and create your new role:

.. image:: images/resized_role_confirmation.png

Creating SecurityMonkey Role
----------------------------

Create a new role and name it "SecurityMonkey":

.. image:: images/resized_name_securitymonkey_role.png

Select "Amazon EC2" under "AWS Service Roles".

.. image:: images/resized_create_role.png

Select "Custom Policy":

.. image:: images/resized_role_policy.png

Paste in this JSON with the name "SecurityMonkeyReadOnly":

.. code-block:: json

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": [
                    "acm:describecertificate",
                    "acm:listcertificates",
                    "cloudtrail:describetrails",
                    "config:describeconfigrules",
                    "config:describeconfigurationrecorders",
                    "directconnect:describeconnections",
                    "ec2:describeaddresses",
                    "ec2:describedhcpoptions",
                    "ec2:describeflowlogs",
                    "ec2:describeimages",
                    "ec2:describeinstances",
                    "ec2:describeinternetgateways",
                    "ec2:describekeypairs",
                    "ec2:describenatgateways",
                    "ec2:describenetworkacls",
                    "ec2:describenetworkinterfaces",
                    "ec2:describeregions",
                    "ec2:describeroutetables",
                    "ec2:describesecuritygroups",
                    "ec2:describesnapshots",
                    "ec2:describesubnets",
                    "ec2:describetags",
                    "ec2:describevolumes",
                    "ec2:describevpcendpoints",
                    "ec2:describevpcpeeringconnections",
                    "ec2:describevpcs",
                    "elasticloadbalancing:describeloadbalancerattributes",
                    "elasticloadbalancing:describeloadbalancerpolicies",
                    "elasticloadbalancing:describeloadbalancers",
                    "es:describeelasticsearchdomainconfig",
                    "es:listdomainnames",
                    "iam:getaccesskeylastused",
                    "iam:getgroup",
                    "iam:getgrouppolicy",
                    "iam:getloginprofile",
                    "iam:getpolicyversion",
                    "iam:getrole",
                    "iam:getrolepolicy",
                    "iam:getservercertificate",
                    "iam:getuser",
                    "iam:getuserpolicy",
                    "iam:listaccesskeys",
                    "iam:listattachedgrouppolicies",
                    "iam:listattachedrolepolicies",
                    "iam:listattacheduserpolicies",
                    "iam:listentitiesforpolicy",
                    "iam:listgrouppolicies",
                    "iam:listgroups",
                    "iam:listinstanceprofilesforrole",
                    "iam:listmfadevices",
                    "iam:listpolicies",
                    "iam:listrolepolicies",
                    "iam:listroles",
                    "iam:listservercertificates",
                    "iam:listsigningcertificates",
                    "iam:listuserpolicies",
                    "iam:listusers",
                    "kms:describekey",
                    "kms:getkeypolicy",
                    "kms:listaliases",
                    "kms:listgrants",
                    "kms:listkeypolicies",
                    "kms:listkeys",
                    "lambda:listfunctions",
                    "rds:describedbclusters",
                    "rds:describedbclustersnapshots",
                    "rds:describedbinstances",
                    "rds:describedbsecuritygroups",
                    "rds:describedbsnapshots",
                    "rds:describedbsubnetgroups",
                    "redshift:describeclusters",
                    "route53:listhostedzones",
                    "route53:listresourcerecordsets",
                    "route53domains:listdomains",
                    "route53domains:getdomaindetail",
                    "s3:getaccelerateconfiguration",
                    "s3:getbucketacl",
                    "s3:getbucketcors",
                    "s3:getbucketlocation",
                    "s3:getbucketlogging",
                    "s3:getbucketnotification",
                    "s3:getbucketpolicy",
                    "s3:getbuckettagging",
                    "s3:getbucketversioning",
                    "s3:getbucketwebsite",
                    "s3:getlifecycleconfiguration",
                    "s3:listallmybuckets",
                    "s3:getreplicationconfiguration",
                    "s3:getanalyticsconfiguration",
                    "s3:getmetricsconfiguration",
                    "s3:getinventoryconfiguration",
                    "ses:getidentityverificationattributes",
                    "ses:listidentities",
                    "ses:listverifiedemailaddresses",
                    "ses:sendemail",
                    "sns:gettopicattributes",
                    "sns:listsubscriptionsbytopic",
                    "sns:listtopics",
                    "sqs:getqueueattributes",
                    "sqs:listqueues"
                ],
                "Effect": "Allow",
                "Resource": "*"
            }
        ]
    }

Review and create the new role.

Allow SecurityMonkeyInstanceProfile to AssumeRole to SecurityMonkey
-------------------------------------------------------------------

You should now have two roles available in your AWS Console:

.. image:: images/resized_both_roles.png

Select the "SecurityMonkey" role and open the "Trust Relationships" tab.

.. image:: images/resized_edit_trust_relationship.png

Edit the Trust Relationship and paste this in:

.. code-block:: json

    {
      "Version": "2008-10-17",
      "Statement": [
        {
          "Sid": "",
          "Effect": "Allow",
          "Principal": {
            "AWS": [
              "arn:aws:iam::<YOUR ACCOUNTID GOES HERE>:role/SecurityMonkeyInstanceProfile"
            ]
          },
          "Action": "sts:AssumeRole"
        }
      ]
    }

Adding more accounts
--------------------

To have your instance of security monkey monitor additional accounts, you must add a SecurityMonkey role in the new account.  Follow the instructions above to create the new SecurityMonkey role.  The Trust Relationship policy should have the account ID of the account where the security monkey instance is running.



**Note**

Additional SecurityMonkeyInstanceProfile roles are not required.  You only need to create a new SecurityMonkey role.

**Note**

You will also need to add the new account in the Web UI, and restart the scheduler.  More information on how do to this will be presented later in this guide.

**TODO**

Document how to setup an SES account and validate it.

Launch an Ubuntu Instance
=========================

Netflix monitors dozens AWS accounts easily on a single m3.large instance.  For this guide, we will launch a m1.small.

In the console, start the process to launch a new Ubuntu instance.  The screenshot below shows EC2 classic, but you can also launch this in external VPC.:

.. image:: images/resized_ubuntu.png

Select an m1.small and select "Next: Configure Instance Details".

**Note: Do not select "Review and Launch".  We need to launch this instance in a specific role.**

.. image:: images/resized_select_ec2_instance.png

Under "IAM Role", select SecurityMonkeyInstanceProfile:

.. image:: images/resized_launch_instance_with_role.png

You may now launch the new instance.  Please take note of the "Public DNS" entry.  We will need that later when configuring security monkey.

.. image:: images/resized_launched_sm.png

Now may also be a good time to edit the "launch-wizard-1" security group to restrict access to your IP.  Make sure you leave TCP 22 open for ssh and TCP 443 for HTTPS.

Keypair
-------

You may be prompted to download a keypair.  You should protect this keypair; it is used to provide ssh access to the new instance.  Put it in a safe place.  You will need to change the permissions on the keypair to 400::

    $ chmod 400 SecurityMonkeyKeypair.pem

Connecting to your new instance:
--------------------------------

We will connect to the new instance over ssh::

    $ ssh -i SecurityMonkeyKeyPair.pem -l ubuntu <PUBLIC_IP_ADDRESS>

Replace the last parameter (<PUBLIC_IP_ADDRESS>) with the Public IP of your instance.

Install Pre-requisites
======================

We now have a fresh install of Ubuntu.  Let's add the hostname to the hosts file::

    $ hostname
    ip-172-30-0-151

Add this to /etc/hosts: (Use nano if you're not familiar with vi.)::

    $ sudo vi /etc/hosts
    127.0.0.1 ip-172-30-0-151

Create the logging folders::

    sudo mkdir /var/log/security_monkey
    sudo chown www-data /var/log/security_monkey
    sudo mkdir /var/www
    sudo chown www-data /var/www
    sudo touch /var/log/security_monkey/security_monkey.error.log
    sudo touch /var/log/security_monkey/security_monkey.access.log
    sudo touch /var/log/security_monkey/security_monkey-deploy.log
    sudo chown www-data /var/log/security_monkey/security_monkey-deploy.log

Let's install the tools we need for Security Monkey::

    $ sudo apt-get update
    $ sudo apt-get -y install python-pip python-dev python-psycopg2 postgresql postgresql-contrib libpq-dev nginx supervisor git libffi-dev 

Setup Postgres
--------------

*For production, you will want to use an AWS RDS Postgres database.*  For this guide, we will setup a database on the instance that was just launched.

First, set a password for the postgres user.  For this guide, we will use ``securitymonkeypassword``: ::

    sudo -u postgres psql
    CREATE DATABASE "secmonkey";
    CREATE ROLE "securitymonkeyuser" LOGIN PASSWORD 'securitymonkeypassword';
    CREATE SCHEMA secmonkey
    GRANT Usage, Create ON SCHEMA "secmonkey" TO "securitymonkeyuser";
    set timezone TO 'GMT';
    select now();
    \q

Clone the Security Monkey Repo
==============================

Next we'll clone and install the package::

    cd /usr/local/src
    sudo git clone --depth 1 --branch master https://github.com/Netflix/security_monkey.git
    cd security_monkey
    sudo python setup.py install

**New in 0.2.0** - Compile the web-app from the Dart code::

    # Get the Google Linux package signing key.
    $ curl https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -

    # Set up the location of the stable repository.
    cd ~
    curl https://storage.googleapis.com/download.dartlang.org/linux/debian/dart_stable.list > dart_stable.list
    sudo mv dart_stable.list /etc/apt/sources.list.d/dart_stable.list
    sudo apt-get update
    sudo apt-get install -y dart

    # Build the Web UI
    cd /usr/local/src/security_monkey/dart
    sudo /usr/lib/dart/bin/pub get
    sudo /usr/lib/dart/bin/pub build

    # Copy the compiled Web UI to the appropriate destination
    sudo /bin/mkdir -p /usr/local/src/security_monkey/security_monkey/static/
    sudo /bin/cp -R /usr/local/src/security_monkey/dart/build/web/* /usr/local/src/security_monkey/security_monkey/static/

Configure the Application
-------------------------

Edit /usr/local/src/security_monkey/env-config/config-deploy.py:

.. code-block:: python

    # Insert any config items here.
    # This will be fed into Flask/SQLAlchemy inside security_monkey/__init__.py

    LOG_CFG = {
        'version': 1,
        'disable_existing_loggers': False,
        'formatters': {
            'standard': {
                'format': '%(asctime)s %(levelname)s: %(message)s '
                    '[in %(pathname)s:%(lineno)d]'
            }
        },
        'handlers': {
            'file': {
                'class': 'logging.handlers.RotatingFileHandler',
                'level': 'DEBUG',
                'formatter': 'standard',
                'filename': '/var/log/security_monkey/securitymonkey.log',
                'maxBytes': 10485760,
                'backupCount': 100,
                'encoding': 'utf8'
            },
            'console': {
                'class': 'logging.StreamHandler',
                'level': 'DEBUG',
                'formatter': 'standard',
                'stream': 'ext://sys.stdout'
            }
        },
        'loggers': {
            'security_monkey': {
                'handlers': ['file', 'console'],
                'level': 'DEBUG'
            },
            'apscheduler': {
                'handlers': ['file', 'console'],
                'level': 'INFO'
            }
        }
    }

    SQLALCHEMY_DATABASE_URI = 'postgresql://securitymonkeyuser:securitymonkeypassword@localhost:5432/secmonkey'

    SQLALCHEMY_POOL_SIZE = 50
    SQLALCHEMY_MAX_OVERFLOW = 15
    ENVIRONMENT = 'ec2'
    USE_ROUTE53 = False
    FQDN = '<PUBLIC_IP_ADDRESS>'
    API_PORT = '5000'
    WEB_PORT = '443'
    FRONTED_BY_NGINX = True
    NGINX_PORT = '443'
    WEB_PATH = '/static/ui.html'
    BASE_URL = 'https://{}/'.format(FQDN)

    SECRET_KEY = '<INSERT_RANDOM_STRING_HERE>'

    MAIL_DEFAULT_SENDER =  'securitymonkey@<YOURDOMAIN>.com'
    SECURITY_REGISTERABLE = True
    SECURITY_CONFIRMABLE = False
    SECURITY_RECOVERABLE = False
    SECURITY_PASSWORD_HASH = 'bcrypt'
    SECURITY_PASSWORD_SALT = '<INSERT_RANDOM_STRING_HERE>'
    SECURITY_TRACKABLE = True

    SECURITY_POST_LOGIN_VIEW = BASE_URL
    SECURITY_POST_REGISTER_VIEW = BASE_URL
    SECURITY_POST_CONFIRM_VIEW = BASE_URL
    SECURITY_POST_RESET_VIEW = BASE_URL
    SECURITY_POST_CHANGE_VIEW = BASE_URL

    # This address gets all change notifications
    SECURITY_TEAM_EMAIL = []

    # These are only required if using SMTP instead of SES
    EMAILS_USE_SMTP = True     # Otherwise, Use SES
    SES_REGION = 'us-east-1'
    MAIL_SERVER = 'smtp.<YOUREMAILPROVIDER>.com'
    MAIL_PORT = 465
    MAIL_USE_SSL = True
    MAIL_USERNAME = 'securitymonkey'
    MAIL_PASSWORD = '<YOURPASSWORD>'

    WTF_CSRF_ENABLED = True
    WTF_CSRF_SSL_STRICT = True # Checks Referer Header. Set to False for API access.
    WTF_CSRF_METHODS = ['DELETE', 'POST', 'PUT', 'PATCH']

    # "NONE", "SUMMARY", or "FULL"
    SECURITYGROUP_INSTANCE_DETAIL = 'FULL'

    # Threads used by the scheduler.
    # You will likely need at least one core thread for every account being monitored.
    CORE_THREADS = 25
    MAX_THREADS = 30

    # SSO SETTINGS:
    ACTIVE_PROVIDERS = []  # "ping", "google" or "onelogin"

    PING_NAME = ''  # Use to override the Ping name in the UI.
    PING_REDIRECT_URI = "{BASE}api/1/auth/ping".format(BASE=BASE_URL)
    PING_CLIENT_ID = ''  # Provided by your administrator
    PING_AUTH_ENDPOINT = ''  # Often something ending in authorization.oauth2
    PING_ACCESS_TOKEN_URL = ''  # Often something ending in token.oauth2
    PING_USER_API_URL = ''  # Often something ending in idp/userinfo.openid
    PING_JWKS_URL = ''  # Often something ending in JWKS
    PING_SECRET = ''  # Provided by your administrator

    GOOGLE_CLIENT_ID = ''
    GOOGLE_AUTH_ENDPOINT = ''
    GOOGLE_SECRET = ''

    ONELOGIN_APP_ID = '<APP_ID>'  # OneLogin App ID provider by your administrator
    ONELOGIN_EMAIL_FIELD = 'User.email'  # SAML attribute used to provide email address
    ONELOGIN_DEFAULT_ROLE = 'View'  # Default RBAC when user doesn't already exist
    ONELOGIN_HTTPS = True  # If using HTTPS strict mode will check the requests are HTTPS
    ONELOGIN_SETTINGS = {
        # If strict is True, then the Python Toolkit will reject unsigned
        # or unencrypted messages if it expects them to be signed or encrypted.
        # Also it will reject the messages if the SAML standard is not strictly
        # followed. Destination, NameId, Conditions ... are validated too.
        "strict": True,

        # Enable debug mode (outputs errors).
        "debug": True,

        # Service Provider Data that we are deploying.
        "sp": {
            # Identifier of the SP entity  (must be a URI)
            "entityId": "{BASE}metadata/".format(BASE=BASE_URL),
            # Specifies info about where and how the <AuthnResponse> message MUST be
            # returned to the requester, in this case our SP.
            "assertionConsumerService": {
                # URL Location where the <Response> from the IdP will be returned
                "url": "{BASE}api/1/auth/onelogin?acs".format(BASE=BASE_URL),
                # SAML protocol binding to be used when returning the <Response>
                # message. OneLogin Toolkit supports this endpoint for the
                # HTTP-POST binding only.
                "binding": "urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"
            },
            # If you need to specify requested attributes, set a
            # attributeConsumingService. nameFormat, attributeValue and
            # friendlyName can be omitted
            #"attributeConsumingService": {
            #        "ServiceName": "SP test",
            #        "serviceDescription": "Test Service",
            #        "requestedAttributes": [
            #            {
            #                "name": "",
            #                "isRequired": False,
            #                "nameFormat": "",
            #                "friendlyName": "",
            #                "attributeValue": ""
            #            }
            #        ]
            #},
            # Specifies info about where and how the <Logout Response> message MUST be
            # returned to the requester, in this case our SP.
            "singleLogoutService": {
                # URL Location where the <Response> from the IdP will be returned
                "url": "{BASE}api/1/auth/onelogin?sls".format(BASE=BASE_URL),
                # SAML protocol binding to be used when returning the <Response>
                # message. OneLogin Toolkit supports the HTTP-Redirect binding
                # only for this endpoint.
                "binding": "urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect"
            },
            # Specifies the constraints on the name identifier to be used to
            # represent the requested subject.
            # Take a look on src/onelogin/saml2/constants.py to see the NameIdFormat that are supported.
            "NameIDFormat": "urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified",
            # Usually x509cert and privateKey of the SP are provided by files placed at
            # the certs folder. But we can also provide them with the following parameters
            "x509cert": "",
            "privateKey": ""
        },

        # Identity Provider Data that we want connected with our SP.
        "idp": {
            # Identifier of the IdP entity  (must be a URI)
            "entityId": "https://app.onelogin.com/saml/metadata/{APP_ID}".format(APP_ID=ONELOGIN_APP_ID),
            # SSO endpoint info of the IdP. (Authentication Request protocol)
            "singleSignOnService": {
                # URL Target of the IdP where the Authentication Request Message
                # will be sent.
                "url": "https://app.onelogin.com/trust/saml2/http-post/sso/{APP_ID}".format(APP_ID=ONELOGIN_APP_ID),
                # SAML protocol binding to be used when returning the <Response>
                # message. OneLogin Toolkit supports the HTTP-Redirect binding
                # only for this endpoint.
                "binding": "urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect"
            },
            # SLO endpoint info of the IdP.
            "singleLogoutService": {
                # URL Location of the IdP where SLO Request will be sent.
                "url": "https://app.onelogin.com/trust/saml2/http-redirect/slo/{APP_ID}".format(APP_ID=ONELOGIN_APP_ID),
                # SAML protocol binding to be used when returning the <Response>
                # message. OneLogin Toolkit supports the HTTP-Redirect binding
                # only for this endpoint.
                "binding": "urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect"
            },
            # Public x509 certificate of the IdP
            "x509cert": "<ONELOGIN_APP_CERT>"
        }
    }

    from datetime import timedelta
    PERMANENT_SESSION_LIFETIME=timedelta(minutes=60)  # Will logout users after period of inactivity.
    SESSION_REFRESH_EACH_REQUEST=True
    SESSION_COOKIE_SECURE=True
    SESSION_COOKIE_HTTPONLY=True
    PREFERRED_URL_SCHEME='https'

    REMEMBER_COOKIE_DURATION=timedelta(minutes=60)  # Can make longer if you want remember_me to be useful
    REMEMBER_COOKIE_SECURE=True
    REMEMBER_COOKIE_HTTPONLY=True

A few things need to be modified in this file before we move on.

**SQLALCHEMY_DATABASE_URI**: The value above will be correct for the username "postgres" with the password "securitymonkeypassword" and the database name of "secmonkey".  Please edit this line if you have created a different database name or username or password.

**FQDN**: You will need to enter the public DNS name you obtained when you launched the security monkey instance.

**SECRET_KEY**: This is used by Flask modules to verify user sessions.  Please use your own random string.  (Keep it secret.)

**SECURITY_CONFIRMABLE**: Leave this off (False) until you have configured and validated an SES account.  More information will be made available on this topic soon.

**SECURITY_RECOVERABLE**: Leave this off (False) until you have configured and validated an SES account.  More information will be made available on this topic soon.

**SECURITY_PASSWORD_SALT**: This is used by flask to salt credentials before putting them into the database.  Please use your own random string.

Other values are self-explanatory.

SECURITY_MONKEY_SETTINGS:
----------------------------------

The SECURITY_MONKEY_SETTINGS environment variable needs to exist and should point to the config-deploy.py we just reviewed.::

    $ export SECURITY_MONKEY_SETTINGS=<Path to your config-deploy.py>

For example::

    $ export SECURITY_MONKEY_SETTINGS=/usr/local/src/security_monkey/env-config/config-deploy.py

Create the database tables:
---------------------------

Security Monkey uses Flask-Migrate (Alembic) to keep database tables up to date.  To create the tables, run  this command::

    cd /usr/local/src/security_monkey/
    sudo -E python manage.py db upgrade

Add Amazon Accounts
==========================
This will add Amazon owned AWS accounts to security monkey. ::

    $ sudo -E python manage.py amazon_accounts

Create the first user:
---------------------------

Users can be created on the command line or by registering in the web UI::

    $ sudo -E python manage.py create_user "you@youremail.com" "Admin"
    > Password:
    > Confirm Password:

create_user takes two parameters.  1) is the email address and 2) is the role.  Roles should be one of these: [View Comment Justify Admin]

Setting up Supervisor
=====================

Supervisor will auto-start security monkey and will auto-restart security monkey if
it were to crash.

.. code-block:: python

    # Control Startup/Shutdown:
    # sudo supervisorctl

    [program:securitymonkey]
    user=www-data

    environment=PYTHONPATH='/usr/local/src/security_monkey/',SECURITY_MONKEY_SETTINGS="/usr/local/src/security_monkey/env-config/config-deploy.py"
    autostart=true
    autorestart=true
    command=python /usr/local/src/security_monkey/manage.py run_api_server

    [program:securitymonkeyscheduler]
    user=www-data
    autostart=true
    autorestart=true
    directory=/usr/local/src/security_monkey/
    environment=PYTHONPATH='/usr/local/src/security_monkey/',SECURITY_MONKEY_SETTINGS="/usr/local/src/security_monkey/env-config/config-deploy.py"
    command=python /usr/local/src/security_monkey/manage.py start_scheduler


Copy /usr/local/src/security_monkey/supervisor/security_monkey.conf to /etc/supervisor/conf.d/security_monkey.conf and make sure it points to the locations where you cloned the security monkey repo.::

    sudo service supervisor restart
    sudo supervisorctl &

Supervisor will attempt to start two python jobs and make sure they are running.  The first job, securitymonkey,
is gunicorn, which it launches by calling manage.py run_api_server.

The second job supervisor runs is the scheduler, which looks for changes every 15 minutes.  **The scheduler will fail to start at this time because there are no accounts for it to monitor**  Later, we will add an account and start the scheduler.

You can track progress by tailing security_monkey-deploy.log.

Create an SSL Certificate
=========================

For this quickstart guide, we will use a self-signed SSL certificate.  In production, you will want to use a certificate that has been signed by a trusted certificate authority.::

    $ cd ~

There are some great instructions for generating a certificate on the Ubuntu website:

`Ubuntu - Create a Self Signed SSL Certificate <https://help.ubuntu.com/12.04/serverguide/certificates-and-security.html>`_

The last commands you need to run from that tutorial are in the "Installing the Certificate" section:

.. code-block:: bash

    sudo cp server.crt /etc/ssl/certs
    sudo cp server.key /etc/ssl/private

Once you have finished the instructions at the link above, and these two files are in your /etc/ssl/certs and /etc/ssl/private, you are ready to move on in this guide.

Setup Nginx:
============

Security Monkey uses gunicorn to serve up content on its internal 127.0.0.1 address.  For better performance, and to offload the work of serving static files, we wrap gunicorn with nginx.  Nginx listens on 0.0.0.0 and proxies some connections to gunicorn for processing and serves up static files quickly.

securitymonkey.conf
-------------------

Save the config file below to: ::

    /etc/nginx/sites-available/securitymonkey.conf

.. code-block:: nginx

    add_header X-Content-Type-Options "nosniff";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Frame-Options "SAMEORIGIN";
    add_header Strict-Transport-Security "max-age=631138519";
    add_header Content-Security-Policy "default-src 'self'; font-src 'self' https://fonts.gstatic.com; script-src 'self' https://ajax.googleapis.com; style-src 'self' https://fonts.googleapis.com;";

    server {
       listen      0.0.0.0:443 ssl;
       ssl_certificate /etc/ssl/certs/server.crt;
       ssl_certificate_key /etc/ssl/private/server.key;
       access_log  /var/log/security_monkey/security_monkey.access.log;
       error_log   /var/log/security_monkey/security_monkey.error.log;

       location ~* ^/(reset|confirm|healthcheck|register|login|logout|api) {
            proxy_read_timeout 120;
            proxy_pass  http://127.0.0.1:5000;
            proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
            proxy_redirect off;
            proxy_buffering off;
            proxy_set_header        Host            $host;
            proxy_set_header        X-Real-IP       $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        location /static {
            rewrite ^/static/(.*)$ /$1 break;
            root /usr/local/src/security_monkey/security_monkey/static;
            index ui.html;
        }

        location / {
            root /usr/local/src/security_monkey/security_monkey/static;
            index ui.html;
        }

    }

Symlink the sites-available file to the sites-enabled folder::

    $ sudo ln -s /etc/nginx/sites-available/securitymonkey.conf /etc/nginx/sites-enabled/securitymonkey.conf

Delete the default configuration::

    $ sudo rm /etc/nginx/sites-enabled/default

Restart nginx::

    $ sudo service nginx restart

Logging into the UI
===================

You should now be able to reach your server

.. image:: images/resized_login_page-1.png

After you have registered a new account and logged in, you need to add an account for Security Monkey to monitor.  Click on "Settings" in the very top menu bar.

.. image:: images/resized_settings_link.png

Adding an Account in the Web UI
-------------------------------

Here you will see a list of the accounts Security Monkey is monitoring.  (It should be empty.)

Click on the plus sign to create a new account:

.. image:: images/empty_settings_page.png

Now we will provide Security Monkey with information about the account you would like to monitor.

.. image:: images/empty_create_account_page.png

When creating a new account in Security Monkey, you may use any "Name" that you would like.  Example names are 'prod', 'test', 'dev', or 'it'.  Names should be unique.

The **S3 Name** has special meaning.  This is the name used on S3 ACL policies.  If you are unsure, it is probably the beginning of the email address that was used to create the AWS account.  (If you signed up as super_geek@example.com, your s3 name is probably super_geek.)  You can edit this value at any time.

The **Number** is the AWS account number.  This must be provided.

**Notes** is an optional field.

**Active** specifies whether Security Monkey should track policies and changes in this account.  There are cases where you want Security Monkey to know about a friendly account, but don't want Security Monkey to track it's changes.

**Third Party** This is a way to tell security monkey that the account is friendly and not owned by you.

**Note: You will need to restart the scheduler whenever you add a new account or disable an existing account.**
We plan to remove this requirement in the future.::

    $ sudo supervisorctl
    securitymonkey                   RUNNING    pid 11401, uptime 0:05:56
    securitymonkeyscheduler          FATAL      Exited too quickly (process log may have details)
    supervisor> start securitymonkeyscheduler
    securitymonkeyscheduler: started
    supervisor> status
    securitymonkey                   RUNNING    pid 11401, uptime 0:06:49
    securitymonkeyscheduler          RUNNING    pid 11519, uptime 0:00:42
    supervisor>

The first run will occur in 15 minutes.  You can monitor all the log files in /var/log/security_monkey/.  In the browser, you can hit the ```AutoRefresh``` button so the browser will attempt to load results every 30 seconds.

**Note: You can also add accounts via the command line with manage.py**::

    $ python manage.py add_account --number 12345678910 --name account_foo
    Successfully added account account_foo

If an account with the same number already exists, this will do nothing, unless you pass ``--force``, in which case, it will override the existing account::

    $ python manage.py add_account --number 12345678910 --name account_foo
    An account with id 12345678910 already exists
    $ python manage.py add_account --number 12345678910 --name account_foo --active false --force
    Successfully added account account_foo

Now What?
=========

Wow. We have accomplished a lot.  Now we can use the Web UI to review our security posture.

Searching in the Web UI
-----------------------

On the Web UI, click the Search button at the top left.  If the scheduler is setup correctly, we should now see items filling the table.  These items are colored if they have issues.  Yellow is for minor issues like friendly cross account access while red indicates more important security issues, like an S3 bucket granting access to "AllUsers" or a security group allowing 0.0.0.0/0.  The newest results are always at the top.

.. image:: images/search_results.png

We can filter these results using the searchbox on the left.  The Region, Tech, Account, and Name fields use auto-complete to help you find what you need.

.. image:: images/filtered_search_1.png

Security Monkey also provides you the ability to search only for issues:

.. image:: images/issues_page.png

Viewing an Item in the Web UI
-----------------------------

Clicking on an item in the web UI brings up the view-item page.

.. image:: images/item_with_issue.png

This item has an attached issue.  Someone has left SSH open to the Internet!  Security Monkey helps you find these types of insecure configurations and correct them.


If Security Monkey finds an issue that you aren't worried about, you should justify the issue and leave a message explaining to others why the configuration is okay.


.. image:: images/justified_issue.png

Security Monkey looks for changes in configurations.  When there is a change, it uses colors to show you the part of the configuration that was affected.  Green tells you that a section was added while red says something has been removed.

.. image:: images/colored_JSON.png

Each revision to an item can have comments attached.  These can explain why a change was made.

.. image:: images/revision_comments.png


Productionalizing Security Monkey
=================================

This guide has been focused on getting Security Monkey up and running quickly.  For a production deployment, you should make a few changes.

Location
--------

Run security_monkey from a separate account.  This will help isolate the instance and the database and ensure the integrity of the change data.

SES
---

Security Monkey uses SES to send email.  While you can install and use Security Monkey without SES, it is recommended that you eventually setup SES to receive Change Reports and Audit Reports.  Enabling SES also allows you to enable the "forgot my password" flow and force users to confirm their email addresses when registering for an account.

To begin the process, you will need to request that AWS enable SES on your account

.. image:: images/SES_LIMITED.png

TODO: Add further documentation on setting up and confirming SES.

RDS
---

In this guide, we setup a postgres database on the instance we launched.  This would be a horrible way to run in production.  You would lose all your data whenever Chaos Monkey unplugged your instance!

Make sure you move your database to an RDS instance. Create a database user with limited permissions and use a different password than the one used in this guide.


Logs
----

If you are relying on security monkey, you really need to ensure that it is running correctly and not hitting a bizarre exception.

Check the Security Monkey logs occasionally.  Let us know if you are seeing exceptions, or better yet, send us a pull request.

Justify Issues
--------------

The daily audit report and the issues-search are most helpful when all the existing issues are worked or justified.  Spend some time to work through the issues found today, so that the ones found tomorrow pop out and catch your attention.

SSL
---

In this guide, we setup a self-signed SSL certificate.  For production, you will want to use a certificate that has been signed by a trusted certificate authority.  You can also attach an SSL cert to an ELB listener.  If so, please use the latest listener reference policy to avoid deprecated ciphers and TLS/SSLv3 attacks.


Ignore List
-------------

If your environment has rapidly changing items that you would prefer not to track in security monkey, please look at the "Ignore List" under the settings page.  You can provide a list of prefixes for each technology, and Security Monkey will ignore those objects when it is inspecting your current AWS configuration.  **Be careful: an attacker could use the ignore list to subvert your monitoring.**

Contribute
----------

It's easy to extend security_monkey with new rules or new technologies.  If you have a good idea, **please send us a pull request**.  I'll be delighted to include them.
