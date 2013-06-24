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

The only changes made to phpmyadmin to enable it to run on CloudFoundry were to configure the database in ```config.inc.php```.  This was done by reading the environment variable VCAP_SERVICES, which is populated by CloudFoundry and contains the connection information for our services, and configuring the host, port, user and password from it.

See this [link](https://github.com/dmikusa-pivotal/cf-ex-phpmyadmin/blob/master/htdocs/config.inc.php#L27) for the details.
