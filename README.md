CloudFoundry PHP example application:  phpmyadmin
=================================================

This is an example application which can be run on CloudFoundry using the PHP Buildpack.

This is an out-of-the-box implementation of phpmyadmin 4.1.7.  It's an example how common PHP applications can easily be run on CloudFoundry.

Usage
-----

Clone the app and push it to CloudFoundry.

```
git clone https://github.com/dmikusa-pivotal/cf-ex-phpmyadmin
cd cf-ex-php-info 
cf push
```

Access your application URL in the browser.

Changes
-------

These changes were made to prepare it to run on CloudFoundry:

1. Configure the database in ```config.inc.php```.  This was done by reading the environment variable VCAP_SERVICES, which is populated by CloudFoundry and contains the connection information for our services, and configuring the host, port from it.  See this [link](https://github.com/dmikusa-pivotal/cf-ex-phpmyadmin/blob/master/htdocs/config.inc.php#L27) for the details.
2. Remove the setup directory, which is not needed.
3. Override the configuration file httpd-directories.conf and prevent access to the libraries directory.  See this [link](https://github.com/dmikusa-pivotal/cf-ex-phpmyadmin/blob/master/config/httpd/extra/httpd-directories.conf#L10) for the details.
4. Set the 'PmaAbsoluteUri' configuration option.  This is needed because the application is using the detected host and port, which are internal to CF, to generate in the URLs.  The links generated with the internal ip / port do not work and so we configure around that by grabbing the application's URL and port. Note that this would not work if multiple URL's were bound to the application.  [Link](https://github.com/dmikusa-pivotal/cf-ex-phpmyadmin/blob/master/htdocs/config.inc.php#L52)
5. Increasedd the timeout of the session by setting 'LoginCookieValidity' and 'session.gc_maxlifetime' to 1800.  Link to [change #1](https://github.com/dmikusa-pivotal/cf-ex-phpmyadmin/blob/master/htdocs/config.inc.php#L56) and [change #2](https://github.com/dmikusa-pivotal/cf-ex-phpmyadmin/blob/master/config/php.ini#L1443).

