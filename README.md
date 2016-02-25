<h1>Introduction</h1>
The purpose of this directory is to provide/document a build process that
can build from source code a LEMP stack, for Ubuntu 15.10, composed of Nginx, MySQL, PHP, and php-fpm.

More specifically, we're going to download the source code and build from scratch, the following packages and versions:
<table>
<tr><td>Package</td><td>Version</td></tr>
<tr><td>MySQL</td><td>5.7.11</tr>
<tr><td>PHP</td><td>5.6.18</tr>
<tr><td>Nginx</td><td>1.9.12</tr>
</table>

This directory contains not merely this README file, but also a handful of initial configuration files.  Hence the version control.

Any installation starts with this directory. During the build, we'll download, unzip, and otherwise create lots of new files in this directory. But even though this directory is under git version control, we don't want to add these new files.

<h2>Rebuild from Source Code</h2>
This process will use .tar.gz (or similar) files containing the source code for all of the above.
We want to be able to very carefully rebuild the same stack, every time, starting from exactly
the same source code.  Attempting to do this using the SCM repos of the various
programs is needlessly complicated because these programs use several different systems.
And attempting to do this using whatever package manager is available on a specific system is
troublesome because the different packages will likely contain subtle differences that will likely cause grief.

<h2>This Stack is Not Entwined With The Rest Of The System</h2>
All of the installation is contained within this directory and uses non-standard ports and user names.  This
stack may thus be easily deleted and rebuilt without affecting other packages on the system.  One reason for using
non-standard ports, in additional to a minor security boost, is to ensure that we really are dealing with this software and not pre-existing
instances that may have been installed previously.

<h2>Install/Execute as Ordinary User</h2>
This installation my be built and executed as an "ordinary" user, ie. not root, and can be placed anywhere
said user has sufficient access rights, such as in his HOME directory.  This will however require that we use
certain non-standard ports.  For example, an ordinary user cannot run a process to bind to port 80.

<h2>Prerequisites</h2>
Although Nginx, MySQL, PHP, and php-fpm can all be installed and executed by an ordinary user,
the host system still needs a variety of build tools.  You will most
conveniently need to sudo or root access to install these tools, if they are not already installed.

Starting from a fresh install of Ubuntu 15.10, I needed to install the following extra packages
to get all this to work:

<table>
<tr><th>Package name</th><th>Why?</th></tr>
<tr><td>cmake</td><td>MySQL</td></tr>
<tr><td>libncurses5-dev</td><td>MySQL <tr><td>libxml2-dev</td><td>PHP</td></tr>
<tr><td>libpcre3-dev</td><td>nginx</td></tr>
MySQL</td></tr>
</table>

Unfortunately, installing all this is outside the scope of this document, so you're on your own with this.

<h2>Rebuild</h2>
Ok, here we go...

<h3>I. Wipe the Slate</h3>
<ol>

<li>
<p>Determine a file-system location for this installation.  Let's refer to that location as STACK_ROOT. In fact, let's export that as an environment variable.</p><p><b>export STACK_ROOT=/home/myhome/lemp</b></p>  <p>Warning: If STACK_ROOT is in a user's home directory, you might be tempted to use the ~ as part of the path.  Resist the urge because
subsequent <b>make install</b> apparently don't like that.</p>
</li>

<li>
p><b>rm -rf $STACK_ROOT</b></p>
<p>Remove any prior installation.</p>
</li>

<li><p><b>mkdir $STACK_ROOT</b></p></li>
<li><p><b>cd $STACK_ROOT</b></p></li>
<li><p><b>git clone https://github.com/bostontrader/ubuntu-nginx-php-mysql.git .</b></p>
<p>Be sure to include the trailing 'dot'  That says to clone the repository with the big awkward name into 'this' directory.</p></li>
</ol>


<h3>II.  Install MySQL 5.6.22</h3>

Let's install MySQL first.  This is the most complicated part and if you can get this working
then the rest of the installation will be easier.  Also, the PHP installation needs MySQL
so we need to install this first.

Since our goal is specifically to build from source code, we'll neglect to use any easier installation methods such as package managers.  A good starting point for building from source is http://dev.mysql.com/doc/refman/5.7/en/source-installation.html

There are a few issues to consider when installing MySQL:

<ul>

<p><li>Which directory will contain the database files?  Not the binaries or configuration, but the database
files themselves.</li></p>

<p><li>How do we carefully control where configuration information comes from?  This can be confusing because
MySQL will routinely look in several places and we can easily mix in config from other installations.</li></p>

<p><li>Which user and group should be the owner of the database files?</li></p>

</ul>

That said, let's doit...

<ol>

<li><p>Earlier I referenced the starting point in the MySQL docs for building the source code.  A more specific reference for our purposes now can be found at:</br>http://dev.mysql.com/doc/refman/5.7/en/installing-source-distribution.html</p></li>

<li><b>export MYSQL_DEFAULT_PORT=3307</b></br><p>Determine a port for the configuration to listen to.  The MySQL default port = 3306 so let's secure by obscurity and use a different port:</br>
</p></li>

<li><b>export MYSQL_USER=batman</b></br><b>export MYSQL_GROUP=catwoman</b></br><p>The access rights for the various files are customarily set according to a user=mysql and a group=mysql.  As with the port, let's again thwart our determined nemeses and use different names.</p></li>

<li>
<p><b>groupadd $MYSQL_GROUP</b></br>
<b>useradd -r -g $MYSQL_GROUP -s /bin/false $MYSQL_USER</b></br>Add the group and user, if necessary.
</p>
</li>

<li><p><b>cd $STACK_ROOT</b></br>Ensure that you're in the STACK_ROOT directory.</p></li>

<li><p><b>wget http://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-boost-5.7.11.tar.gz</b></p><p>This downloads the source code, including some c++ boost source/headers.</p></li>

<li><p><b>tar xvf mysql-boost-5.7.11.tar.gz</b></p>
</p></li>

<li><p><b>cd $STACK_ROOT/mysql-5.7.11</b></p>
</p></li>

<li><p><b>cmake -L</b></br>
Optional.  Gives a brief overview of important configuration parameters. You can change their values
by using the -D option.  See supra for example.</p>
</li>

<li><p><b>cmake . -DCMAKE_INSTALL_PREFIX=$STACK_ROOT/mysql -DMYSQL_DATADIR=$STACK_ROOT/mysql/data -DWITH_BOOST=$STACK_ROOT/mysql-5.7.11/boost</b>

</p></li>

<li><p>If you want to start over with cmake, then do <b>rm CMakeCache.txt</b></p></li>

<li><p><b>make</b></p></li>

<li><p><b>make test</b></p></li>

<li><p><b>make install</b></p></li>
</ol>

<h4>Post installation</h4>
<ol>
<li><p><b>cd $STACK_ROOT/mysql</b></p></li>
<li><p><b>sudo chown -R $MYSQL_USER .</b></p></li>
<li><p><b>sudo chgrp -R $MYSQL_GROUP .</b></p></li>
<li><p><b>sudo bin/mysqld --initialize --user=$MYSQL_USER</b></p><p>This may take a few moments, so please be patient.</p></li>
<li><p>The output from the prior step includes a temporary password for root@localhost.  Be sure to write this down!</p></li>

<li><p><b>cd $STACK_ROOT/mysql</b></p>
<p><b>sudo mkdir mysql-files</b></p>
<p><b>sudo chmod 750 mysql-files</b></p>
<p><b>sudo chown -R $MYSQL_USER .</b></p>
<p><b>sudo chgrp -R $MYSQL_GROUP .</b></p>
</ol>

<h4>Post Installation and Initialization</h4>
<p>At this point the MySQL daemon and various utilities are ready to run.  Although the installation created
a default conf file, we don't want to use it.  Instead, we'll feed the binaries whatever options via command line.
The following options are typically used for our application:

<ul>
<li><p><b>--no-defaults</b>  If we don't carefully suppress the use of any
default configuration files, MySQL will diligently try to find some configuration and it may find
and use configuration from some other installation.  So we generally want to use this option to prevent that.</p>
</li>
<li><p><b>--user=$MYSQL_USER</b> The actual db files are owned by this user.  If we don't specify this then
MySQL will typically use the currently logged in user.</p></li>
<li><p><b>--port=MYSQL_DEFAULT_PORT</b> Which TCP/IP port will the program listen on?</p></li>
<li><p><b>--protocol=tcp</b> Sometimes this is required.  Sometimes the use of the --port option apparently implies this option as well.</p></li>
<li><p><b>--datadir=?</b> Where are the datafiles?</p></li>
<li><p><b>--log-error=?</b> Where is the error log?</p></li>
</ul>

That said... let's crank it up!
<p></br><b>sudo $STACK_ROOT/mysql/bin/mysqld_safe
 --user=$MYSQL_USER
 --log-error=$STACK_ROOT/mysql.err --datadir=$STACK_ROOT/mysql/data
 --port=$MYSQL_DEFAULT_PORT</b></p>

<p>mysqld_safe is the preferred way to execute mysqld.  This command will _not_ be daemonized and will consume your terminal window.  So append the "&" character to daemonize or open another terminal window for subsequent work.</p>

<h4>Verify basic installation and operation</h4>

<b>sudo ps -A | grep mysqld</b></br>
Verify that mysql is a process.
You should see both "mysqld" and "mysqld_safe"

<b>netstat -lnp  | grep mysql</b></br>
Verify that mysql is listening on the expected port.
Do you see $MYSQL_DEFAULT_PORT and "LISTEN" in there somewhere?

<b>killall -r mysql</b></br>
Stop the server, ending any processes it had.


<h4>Review the directory structure, relevant to MySQL</h4>

<b>$STACK_ROOT/mysql-5.7.11.tar.gz</b> - This is the original installation media.  

<b>$STACK_ROOT/mysql-5.7.11</b> - This is the installation source code as extracted from the above.

<b>$STACK_ROOT/mysql</b> - This contains the MySQL installation that was built from the above.

<h3>III. Install PHP</h3>

Now install PHP 5.6.18.  This release also includes php-fpm so that will be installed as well.
We will however configure php-fpm separately.

<ol>
<li><p><b>cd $STACK_ROOT</b></br>
Ensure that you're in the STACK_ROOT/ubuntu-nginx-php-mysql directory.</p></li>

<li><p><b>wget http://hk1.php.net/distributions/php-5.6.18.tar.bz2</b></p></li>

<li><p><b>tar xvf php-5.6.18.tar.bz2</b></p></li>

<li><p><b>cd $STACK_ROOT/php-5.6.18</b></p></li>

<li><p><b>./configure --help</b></br>This is optional but possibly useful. Possibly review the modules being loaded and exclude some of them.</p></li>

<li><p><b>./configure --prefix=$STACK_ROOT/php --enable-fpm --with-mysql=$STACK_ROOT/mysql</b></p></li>

We need fpm!

<li><p><b>make --help</b> Optional.</p></li>

<li><p><b>make</b></p></li>

<li><p><b>make test</b>  This may reveal some minor errors.  Don't worry about them.</p></li>

<li><p><b>make install</b></p></li>
</ol>

Note: After installation, there is no initial php.ini.  This will be added later.

<h4>Verify basic installation and operation of PHP</h4>

<b>$STACK_ROOT/php/bin/php --version</b></br>
Look for version or info about php.
  Does this say 5.6.18?

<b>$STACK_ROOT/php/bin/php --info</b></br>Look at the info....

<b>STACK_ROOT/php/bin/php --info | grep "Loaded Configuration"</b></br>
Narrow the search for "Loaded Configuration".  Is php using the php.ini we expect?
At this point, there should be none loaded at all.

<h4>Review the directory structure, relevant to PHP</h4>

<b>$STACK_ROOT/php-5.6.5.tar.bz2</b> - This is the original installation media.  Not under SCM.

<b>$STACK_ROOT/php-5.6.5</b> - This is the installation source code as extracted from the above.
Not under SCM.

<b>$STACK_ROOT/php</b> - This contains the PHP installation that was built from the above. Not under SCM.


<h3>IV. Install php-fpm.</h3>

php-fpm has already been installed via our installation of PHP.  The versioning is not relevant because it's
whatever version comes with PHP 5.6.18.  We do however need to configure php-fpm and learn a bit about its
ways and customs.

Determine a port for the initial configuration to listen to.  By default it's port 9000.  For this example
let's use 9001.

PHPFMP_DEFAULT_PORT = 9001

Use a custom built configuration provided by this project.

The stock configuration is filled with commented out examples.  This just confuses everything.
The custom built config has _nothing_ except things we specifically want.  We'll otherwise just rely on
the default operation of php-fpm until and unless we specifically decide otherwise.

<b>ln -s $STACK_ROOT/php-fpm-conf/php-fpm.conf $STACK_ROOT/php/etc/php-fpm.conf</b>


<h4>Verify basic installation and operation</h4>

Help, version, info.

<b>$STACK_ROOT/php/sbin/php-fpm --help</b></br>
<b>$STACK_ROOT/php/sbin/php-fpm -v</b></br>
<b>$STACK_ROOT/php/sbin/php-fpm -i</b>

<b>$STACK_ROOT/php/sbin/php-fpm -t</b>
</br>Test the configuration file.


<b>STACK_ROOT/php/sbin/php-fpm</b></br>This will start the php-fpm server.

<b>netstat -lnp  | grep php-fpm</b></br>
Verify that php-fpm is listening on the expected port.

Do you see "127.0.0.1:PHP-PFM_DEFAULT_PORT" and "LISTEN"?

<b>ps -A | grep php-fpm</b></br>
<b>killall php-fpm</b></br>

Stop the php-fpm server.

<h4>Review the directory structure, relevant to php-fpm</h4>

Recall that php-fpm was installed with PHP so therefore
php-fpm will not have it's own original installation media or extracted source or installation directory.

<b>$STACK_ROOT/php-fpm-conf</b></br>
This contains the versioned configuration files that we develop.

<b>$STACK_ROOT/php/sbin/php-fpm</b></br>
This is the executable binary.

<b>$STACK_ROOT/php/etc/php-fpm.conf</b></br>
This is a link to the configuration file.


<h3>V. Install nginx</h3>

Now it's time to install nginx version 1.9.12.

<ol>
<li><b>export $NGINX_DEFAULT_PORT=3000</b></br><p>Determine a port for the initial configuration to listen to.  As an unprivileged user we cannot use port 80.</p></li>

<li><p><b>cd $STACK_ROOT</b></li>

<li><p><b>wget http://nginx.org/download/nginx-1.9.12.tar.gz</b></li>
<li><p><b>tar xvf nginx-1.9.12.tar.gz</b></br></p></li>
<li><p><b>cd $STACK_ROOT/nginx-1.9.12</b></br></p></li>

<li><p><b>./configure --help</b></br>
  This is not strictly necessary, but it may be useful. Possibly review the modules being loaded and exclude some of them.</p></li>

  <li><p><b>./configure --prefix=$STACK_ROOT/nginx --without-http_gzip_module</b>
  <br>Look at the output of configure.  It will tell you that the binaries, the configuration, the logs, etc. are all found inside this directory.</p>

  <p>We don't need gzip compression right now, which requires zlib, and it's easier to omit gzip
   than to install zlib at this time.</p>
  </li>

  <li><p><b>make --help</b></br>Optional, but may be useful.</p></li>

  <li><p>
  <b>make</b></br>
  <b>make install</b></br></p></li>



  <li><p><b></b></br></p></li>







<li><b>make --help</b>  Again, this is optional, but may be useful.</li>

</li><b>make</b></li>

<li><b>make install</b></li>

<li>Replace the installed configuration directory with the custom built configuration provided
by this project.</li>

<li><b>rm -rf STACK_ROOT/ubuntu-nginx-php-mysql/nginx/conf/*</b>  Remove the contents of the existing directory.</li>

<li><b>ln -s STACK_ROOT/ubuntu-nginx-php-mysql/nginx-conf/nginx.conf STACK_ROOT/ubuntu-nginx-php-mysql/nginx/conf/nginx.conf</b>  Link to new config</li>

The stock configuration is filled with commented out examples.  This just confuses everything.
The custom built config has _nothing_ except things we specifically want.  We'll just rely on
the default operation of nginx until and unless we specifically decide otherwise.
</li>

<li>Verify basic installation and operation:

Test the configuration file:
<b>STACK_ROOT/ubuntu-nginx-php-mysql/nginx/sbin/nginx -t</b>

This will start the server.
<b>STACK_ROOT/ubuntu-nginx-php-mysql/nginx/sbin/nginx &</b>

Recall that the "&" symbol makes this command run as a daemon.

Verify that nginx is listening on the expected port:
<b>netstat -lnp  | grep "nginx"</b>
Do you see "0 0.0.0.0:NGINX_DEFAULT_PORT" and "LISTEN"?

From your browser of choice, view this server, using the NGINX_DEFAULT_PORT.  For example,
what's the IP address of the server?  Browse to ip-address:NGINX_DEFAULT_PORT.  Do you see the nginx
welcome message? As an exercise, find this message and modify it in order to verify that you really
understand where this is being served from.

Restart the server and reload the config.
<b>STACK_ROOT/ubuntu-nginx-php-mysql/nginx/sbin/nginx -s reload</b>

Display any processes running nginx.  There should be two processes, unless some other nginx
is running on this server.
<b>ps -A | grep "nginx"</b>

Stop the server, ending any processes it had.
<b>STACK_ROOT/ubuntu-nginx-php-mysql/nginx/sbin/nginx -s stop</b>
</li>

<li>Review the directory structure, relevant to nginx.

<b>STACK_ROOT/ubuntu-nginx-php-mysql/nginx-1.7.9.tar.gz</b> - This is the original installation media.  Not under SCM.

<b>STACK_ROOT/ubuntu-nginx-php-mysql/nginx-1.7.9</b> - This is the installation source code as extracted from the above.
Not under SCM.  Note: this also contains the original configuration.

<b>STACK_ROOT/ubuntu-nginx-php-mysql/nginx</b> - This contains the nginx installation that was built from the above,
which is the binaries, configuration, log files, and html to serve.  Not under SCM.

<b>STACK_ROOT/ubuntu-nginx-php-mysql/nginx-conf</b> - This contains the versioned configuration files that we develop, that are later copied into the STACK_ROOT/ubuntu-nginx-php-mysql/nginx/conf directory.
</li>
</ol>

<h3>VI. phpinfo() via PHP & Nginx</h3>

Now it's time to modify our configuration so that we can view a .php file, that contains a call to phpinfo()
and view this in the browser, via nginx.  Since we're using php-fpm, we'll have to get that properly configured as well.

1. Replace the installed nginx configuration directory with the custom built configuration provided
by this project.  Note: This is the 2nd custom config that we're using.

<li><b>ln -sf STACK_ROOT/ubuntu-nginx-php-mysql/nginx-conf/nginx.conf2 STACK_ROOT/ubuntu-nginx-php-mysql/nginx/conf/nginx.conf</b>  Link to new config.  This time you'll
want to force (f) the link to replace the existing link.</li>

<li>Edit nginx/conf/nginx.conf to replace any references to STACK_ROOT with the actual value.</li>

2. Turn on (or reload) nginx.

3. Turn on php-fpm.

4. From your browser of choice, navigate to localhost:NGINX_DEFAULT_PORT/phpinfo.php.  Do you see the php info message?
