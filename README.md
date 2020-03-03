## CloudFoundry PHP Example Application:  PHPMyAdmin

This is an example application which can be run on CloudFoundry using the [PHP Build Pack].

This is an out-of-the-box implementation of PHPMyAdmin. It's an example how common PHP applications can easily be run on CloudFoundry.

### Usage

1. Download the latest phpMyAdmin from [the project site](https://www.phpmyadmin.net/downloads/).

1. Extract all of the files from phpMyAdmin to the `htdocs` directory. For example: `tar -zx --strip-components=1 -C htdocs/ -f ~/Downloads/phpMyAdmin-5.0.1-english.tar.gz`. When done, you should have an `htdocs` directory that looks like this...

  ```bash
  drwxr-xr-x  114 piccolo  wheel    3648 Mar  3 10:59 .
  drwxr-xr-x   11 piccolo  wheel     352 Mar  3 10:55 ..
  -rw-r--r--@   1 piccolo  wheel    3216 Jan  7 21:04 CODE_OF_CONDUCT.md
  -rw-r--r--@   1 piccolo  wheel    2592 Jan  7 21:04 CONTRIBUTING.md
  -rw-r--r--@   1 piccolo  wheel   21411 Jan  7 21:04 ChangeLog
  -rw-r--r--@   1 piccolo  wheel    1816 Jan  7 21:04 DCO
  -rw-r--r--@   1 piccolo  wheel   18092 Jan  7 21:04 LICENSE
  -rw-r--r--@   1 piccolo  wheel    1520 Jan  7 21:04 README
  -rw-r--r--@   1 piccolo  wheel      29 Jan  7 21:04 RELEASE-DATE-5.0.1
  -rw-r--r--@   1 piccolo  wheel    2011 Jan  7 21:04 ajax.php
  -rw-r--r--@   1 piccolo  wheel    1815 Jan  7 21:04 browse_foreigners.php
  ...
  ```

1. If you don't have one already, create a MySQL service.  With Pivotal Web Services, the following command will create a free MySQL database through [ClearDb].

   ```bash
   cf create-service cleardb spark mysql
   ```
   *If you have an existing DB service with some arbitary name, please re-name the service such that it contains the string `mysql`. The `config.inc.php` file will automatically configure MySQL services if the plan is ClearDb or Pivotal MySQL, if they are tagged with `mysql` or if the service name contains the word `mysql`. If you do not follow this pattern, your MySQL service will require manual configuration.*

1. Copy `htdocs/config.sample.inc.php` to `htdocs/config.inc.php` & edit `htdocs/config.inc.php`. First, comment out the lines below `First server` and then insert the following directly below the link `$i = 0;`.

  ```php
  /*
  * Read MySQL service properties from 'VCAP_SERVICES' env variable
  */
  $service_blob = json_decode(getenv('VCAP_SERVICES'), true);
  $mysql_services = array();
  foreach($service_blob as $service_provider => $service_list) {
      // looks for 'cleardb' or 'p-mysql' service
      if ($service_provider === 'cleardb' || $service_provider === 'p-mysql') {
          foreach($service_list as $mysql_service) {
              $mysql_services[] = $mysql_service;
          }
          continue;
      }
      foreach ($service_list as $some_service) {
          // looks for tags of 'mysql'
          if (in_array('mysql', $some_service['tags'], true)) {
              $mysql_services[] = $some_service;
              continue;
          }
          // look for a service where the name includes 'mysql'
          if (strpos($some_service['name'], 'mysql') !== false) {
              $mysql_services[] = $some_service;
          }
      }
  }

  /*
  * Servers configuration
  */
  for ($i = 1; $i <= count($mysql_services); $i++) {
      $db = $mysql_services[$i-1]['credentials'];
      /* Display name */
      $cfg['Servers'][$i]['verbose'] = $mysql_services[$i-1]['name'];
      /* Authentication type */
      $cfg['Servers'][$i]['auth_type'] = 'cookie';
      /* Server parameters */
      $cfg['Servers'][$i]['host'] = $db['hostname'];
      $cfg['Servers'][$i]['port'] = $db['port'];
      $cfg['Servers'][$i]['connect_type'] = 'tcp';
      $cfg['Servers'][$i]['compress'] = false;
      $cfg['Servers'][$i]['extension'] = 'mysqli';
      $cfg['Servers'][$i]['AllowNoPassword'] = false;
  }

  /* force traffic to use HTTPS */
  $appCfg = json_decode(getenv('VCAP_APPLICATION'), true);
  $scheme = ($_SERVER['HTTPS'] != '') ? 'https' : 'http';
  $cfg['PmaAbsoluteUri'] = $scheme . '://' . $appCfg['uris'][0] . "/";
  $cfg['LoginCookieValidity'] = 1440;
  $cfg['ForceSSL'] = true;
  ```

  Second, look for the config line at the top `$cfg['blowfish_secret']`. Insert a random 32 character passphrase as the value for that option. Save and close the file.

1. Edit the `manifest.yml` file. Insert your service name, if it's not `mysql`. Also, set the route that you would like to use.

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
