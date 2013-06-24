CloudFoundry PHP example application:  phpmyadmin
=================================================

This is an example application which can be run on CloudFoundry using the PHP Buildpack.

This is an out-of-the-box implementation of phpmyadmin 4.0.4.  It's an example how common PHP applications can easily be run on CloudFoundry.

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
3. Override the configuration file httpd-directories.conf and prevent access to the libraries directory.  See this [link](https://github.com/dmikusa-pivotal/cf-ex-phpmyadmin/blob/master/config/extras/httpd-directories.conf) for the details.
4. Set the 'PmaAbsoluteUri' configuration option.  This is needed because the application is using the detected host and port and setting it in the URL returned in the Location header after redirects.  This is bad because it's using an internal port and not 80 and CloudFoundry is not rewriting the Location header.  This works around the issue.
5. Increasedd the timeout of the session by setting 'LoginCookieValidity' and 'session.gc_maxlifetime' to 1800.

