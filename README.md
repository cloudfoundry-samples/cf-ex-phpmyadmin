## CloudFoundry PHP Example Application:  PHPMyAdmin

This is an example application which can be run on CloudFoundry using the [PHP Build Pack].

This is an out-of-the-box implementation of PHPMyAdmin 4.2.2.  It's an example how common PHP applications can easily be run on CloudFoundry.

### Usage

1. Clone the app (i.e. this repo).

  ```bash
  git clone https://github.com/cloudfoundry-samples/cf-ex-phpmyadmin
  cd cf-ex-phpmyadmin
  ```

1. If you don't have one already, create a MySQL service.  With Pivotal Web Services, the following command will create a free MySQL database through [ClearDb].

  ```bash
  cf create-service cleardb spark mysql
  ```

1. Push it to CloudFoundry.

  ```bash
  cf push
  ```

  Access your application URL in the browser.  Login with the credentials for your service.  If you need to find these, just run this command and look for the VCAP_SERVICES environment variable under the `System Provided` section.

  ```bash
  cf env <app-name>
  ```

### How It Works

When you push the application here's what happens.

1. The local bits are pushed to your target.  This is small, six files around 30k. It includes the changes we made and a build pack extension for PHPMyAdmin.
1. The server downloads the [PHP Build Pack] and runs it.  This installs HTTPD and PHP.
1. The build pack sees the extension that we pushed and runs it.  The extension downloads the stock PHPMyAdmin file from their server, unzips it and installs it into the `htdocs` directory.  It then copies the rest of the files that we pushed and replaces the default PHPMyAdmin files with them.  In this case, it's just the `config.inc.php` file.
1. At this point, the build pack is done and CF runs our droplet.

### Changes

These changes were made to prepare it to run on CloudFoundry:

1. Configure the database in `config.inc.php`.  This was done by reading the environment variable VCAP_SERVICES, which is populated by CloudFoundry and contains the connection information for our services, and configuring the host, port from it.  See this [link](https://github.com/cloudfoundry-samples/cf-ex-phpmyadmin/blob/master/htdocs/config.inc.php#L27) for the details.
2. Remove the setup directory, which is not needed.
3. Override the configuration file httpd-directories.conf and prevent access to the libraries directory.  See this [link](https://github.com/cloudfoundry-samples/cf-ex-phpmyadmin/blob/master/.bp-config/httpd/extra/httpd-directories.conf#L14) for the details.
4. Set the 'PmaAbsoluteUri' configuration option.  This is needed because the application is using the detected host and port, which are internal to CF, to generate in the URLs.  The links generated with the internal ip / port do not work and so we configure around that by grabbing the application's URL and port. Note that this would not work if multiple URL's were bound to the application.  [Link](https://github.com/cloudfoundry-samples/cf-ex-phpmyadmin/blob/master/htdocs/config.inc.php#L52)
5. Increased the timeout of the session by setting 'LoginCookieValidity' and 'session.gc_maxlifetime' to 1800.  Link to [change #1](https://github.com/cloudfoundry-samples/cf-ex-phpmyadmin/blob/master/htdocs/config.inc.php#L56) and [change #2](https://github.com/cloudfoundry-samples/cf-ex-phpmyadmin/blob/master/.bp-config/php/php.ini#L1443).

[PHP Buildpack]:https://github.com/cloudfoundry/php-buildpack
[ClearDb]:https://www.cleardb.com/
