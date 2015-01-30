<h1>Introduction</h1>
The purpose of this directory is to provide/document a build process that
can build from scratch a stack, for Ubuntu 14, composed of nginx, php, php-fpm, and mysql.

<h2>Rebuild from Source Code</h2>
This process will use .tar.gz (or similar) files containing the source code for all of the above.
We want to be able to very carefully rebuild the same stack, every time, starting from exactly 
the same source code.  Attempting to do this using the SCM repos of the various
programs is needlessly complicated because these programs use several different systems.
And attempting to do this using whatever package manager is available on a specific system is 
troublesome because the different packages will likely contain subtle differences that may introduce bugs.

<h2>This Stack is Not Entwined With The Rest Of The System</h2>
All of the installation is contained within this directory and uses non-standard ports and user names.  This
stack may thus be easily deleted and rebuilt without affecting other packages on the system.  One reason for using
non-standard ports is to ensure that we really are dealing with this software and not pre-existing
instances that may have been installed previously.

<h2>Install/Execute as Ordinary User</h2>
This installation my be built and executed as an "ordinary" user, ie. not root, and can be placed anywhere
said user has sufficient access rights, such as in his HOME directory.  This will however require that we use
certain non-standard ports.  For example, an ordinary user cannot run a process to bind to port 80.

<h2>Prerequisites</h2>
Although nginx, php, php-fpm, and mysql can all be installed and executed as an ordinary user,
the host system still needs a variety of tools such as build tools and compilers.  You will most
conveniently need to sudo or root access to install these tools, if they are not already installed.

Starting from a fresh install of Ubuntu 14.04 LTS, I needed to install the following extra packages
to get all this to work:

build-essential
cmake
git-core
g++
libncurses5
libpcre3-dev - nginx
libxml2-dev  - php

<h2>Rebuild</h2>
Ok, here we go...

<h3>I. Wipe the Slate</h3>
<ol>

<li>
Determine a file-system location for this installation.  Let's refer to that location as STACK_ROOT.  Warning:
I tried this once using a user's home directory as the STACK_ROOT, which I referred to using the "~".
But make install didn't install anything.  So I tried again, using the same directory, but this time
using the absolute path instead.  This worked.
</li>

<li><b>cd STACK_ROOT</b></li>

<li>
Remove any prior installation, if desired.<br>
<b>rm -rf STACK_ROOT/ubuntu-nginx-php-mysql</b>.<br>
This command will remove this directory, all files and subdirectories within (r) and force it without any prompts (f).
</li>

<li><b>git clone https://github.com/bostontrader/ubuntu-nginx-php-mysql.git</b></li>

<li><b>cd STACK_ROOT/ubuntu-nginx-php-mysql</b></li>
</ol>


<h3>II.  Install MySQL 5.6.22</h3>

Let's install MySQL first.  This is the most complicated part and if you can get this working
then the rest of the installation will be easier.  Also, the PHP installation needs MySQL
so we need to install this first.

There are a few issues to consider when installing MySQL:

<ul>

<li>Which directory will contain the database files?  Not the binaries or configuration, but the database
files themselves.</li>

<li>How do we carefully control where configuration information comes from?  This can be confusing because
MySQL will routinely look in several places and we can easily mix in config from other installations.</li>

<li>Which user should be the owner of the database files?</li>

</ul>

That said, let's doit...

<ol>

<li>Determine a port for the configuration to listen to.  The MySQL default port = 3306 so in this example we'll use
MYSQL_DEFAULT_PORT = 3307.</li>

<li>Designate a particular user to be the owner of the db files.  DB_USER=<whatever>.</li>

<li>Ensure that you're in the STACK_ROOT/ubuntu-nginx-php-mysql directory.</li>

<li><b>wget http://cdn.mysql.com/Downloads/MySQL-5.6/mysql-5.6.22.tar.gz</b></li>

<li><b>tar -xvf mysql-5.6.22.tar.gz</b></li>

<li><b>cd mysql-5.6.22</b></li>

<li><b>cmake -L</b>
Optional.  Gives a brief overview of important configuration parameters. You can change their values
by using the -D option.  See supra for example.
</li>

<li><b>cmake . -DCMAKE_INSTALL_PREFIX=STACK_ROOT/ubuntu-nginx-php-mysql/mysql -DMYSQL_DATADIR=STACK_ROOT/ubuntu-nginx-php-mysql/mysql</b>

Note: It's probably best to not use a ~ in STACK_ROOT.  Maybe that would work but
why taunt fate?  Feel free to figure this out at your leisure.</li>

<li>If you want to start over with cmake, then do <b>rm CMakeCache.txt</b></li>

<li><b>make</b></li>

<li><b>make test</b></li>

<li><b>make install</b></li>

<li>
Now we need to install the beginning db that MySQL itself needs in order to function.  For example, the
grant tables. 

<b>STACK_ROOT/ubuntu-nginx-php-mysql/mysql/scripts/mysql_install_db --no-defaults --datadir=STACK_ROOT/ubuntu-nginx-php-mysql/mysql/data  --srcdir=STACK_ROOT/ubuntu-nginx-php-mysql/mysql-5.6.22</b>

The --srcdir option tells mysql_install_db to use the binaries that we've built earlier, not the ones that have been "installed"
somewhere.  This saves us the trouble of setting the PATH to these binaries.

Please be patient, this takes a few moments to run and will eventually terminate on its own.

After this command has been run, it will display some useful information about how to start and secure the mysql server.
Read this and compare to these notes.

Also a warning that there's no root password and other security info.

</li>

<li>At this point the MySQL daemon and various utilities are ready to run.  Although the installation created
a default conf file, we don't want to use it.  Instead, we'll feed the binaries whatever options via command line.
The following options are typically used for our application:

<ul>
<li><b>--no-defaults</b>  If we don't carefully suppress the use of any 
default configuration files, MarieDb will diligently try to find some configuration and it may find
and use configuration from some other installation.  So we generally want to use this option to prevent that.
</li>
<li><b>--user=DB_USER</b> The actual db files are owned by this user.  If we don't specify this then
mysql will typically use the currently logged in user.</li>
<li><b>--port=MYSQL_DEFAULT_PORT</b> Which TCP/IP port will the program listen on?</li>
<li><b>--protocol=tcp</b> Sometimes this is required.  Sometimes the use of the --port option apparently implies this option as well.</li>
<li><b>--datadir=?</b> Where are the datafiles?</li>
<li><b>--log-error=?</b> Where is the error log?</li>
</ul>

</li>

<li>That said... turn on the MySQL server.
<b>STACK_ROOT/ubuntu-nginx-php-mysql/mysql/bin/mysqld_safe --no-defaults 
  --log-error=STACK_ROOT/ubuntu-nginx-php-mysql --datadir=STACK_ROOT/ubuntu-nginx-php-mysql/mysql/data 
  --port=3307</b>

Note: No --protocol option used.  Must be implied because of the --port option.

mysqld_safe is the preferred way to execute mysqld.  This command will _not_ be daemonized and will consume your terminal window.  So append the "&" character to daemonize or open another terminal window for subsequent work.
</li>

<li>Verify basic installation and operation:

Verify that mysql is a process:

<b>ps -A | grep "mysqld"</b>

You should see both "mysqld" and "mysqld_safe"

Verify that mysql is listening on the expected port:
<b>netstat -lnp  | grep "mysql"</b>
Do you see "MYSQL_DEFAULT_PORT" and "LISTEN" in there somewhere?

Stop the server, ending any processes it had.

<b>killall mysqld</b>
</li>

<li>Review the directory structure, relevant to mysql.

<b>STACK_ROOT/ubuntu-nginx-php-mysql/mysql-5.6.22.tar.gz</b> - This is the original installation media.  Not under SCM.

<b>STACK_ROOT/ubuntu-nginx-php-mysql/mysql-5.6.22</b> - This is the installation source code as extracted from the above. 
Not under SCM.

<b>STACK_ROOT/ubuntu-nginx-php-mysql/mysql</b> - This contains the MySQL installation that was built from the above. 
Not under SCM.

</li>

</ol>

<h3>III. Install PHP</h3>

Now install PHP 5.6.5.  This release also includes php-fpm so that will be installed as well.
We will however configure php-fpm separately.

1. <b>Ensure that you're in the STACK_ROOT/ubuntu-nginx-php-mysql directory.</b>

2. <b>wget http://hk1.php.net/distributions/php-5.6.5.tar.bz2</b>

3. <b>tar -xvf php-5.6.5.tar.bz2</b>

4. <b>cd php-5.6.5</b>

5. <b>./configure --help</b>  This is optional but possibly useful. Possibly review the modules being loaded and exclude some of them.

6. <b>./configure --prefix=STACK_ROOT/ubuntu-nginx-php-mysql/php --enable-fpm --with-mysql=STACK_ROOT/ubuntu-nginx-php-mysql/mysql</b>

We need fpm!

7. <b>make --help</b> Optional.

8. <b>make</b>

9. <b>make test</b>  This may reveal some minor errors.  Don't worry about them.

10. <b>make install</b>

Note: After installation, there is no initial php.ini.  This will be added later.

11. Verify basic installation and operation of php.

Look for version or info about php.
<p>STACK_ROOT/ubuntu-nginx-php-mysql/php/bin/php --version</b>  Does this say 5.6.5?

Look at the info....
<p>STACK_ROOT/ubuntu-nginx-php-mysql/php/bin/php --info</b>

Narrow the search for "Loaded Configuration".  Is php using the php.ini we expect?
At this point, there should be none loaded at all.
<p>STACK_ROOT/ubuntu-nginx-php-mysql/php/bin/php --info | grep "Loaded Configuration"</b>

12. Review the directory structure, relevant to php.

<b>STACK_ROOT/ubuntu-nginx-php-mysql/php-5.6.5.tar.bz2</b> - This is the original installation media.  Not under SCM.

<b>STACK_ROOT/ubuntu-nginx-php-mysql/php-5.6.5</b> - This is the installation source code as extracted from the above.
Not under SCM.

<b>STACK_ROOT/ubuntu-nginx-php-mysql/php</b> - This contains the php installation that was built from the above. Not under SCM.


<h3>IV. Install php-fpm.</h3>

php-fpm has already been installed via our installation of php.  The versioning is not relevant because it's
whatever version comes with PHP 5.6.5.  We do however need to configure php-fpm and learn a bit about its 
ways and customs.

1. Determine a port for the initial configuration to listen to.  By default it's port 9000.  For this example
let's use 9001.

PHPFMP_DEFAULT_PORT = 9001

2. Use a custom built configuration provided by this project.

The stock configuration is filled with commented out examples.  This just confuses everything.
The custom built config has _nothing_ except things we specifically want.  We'll otherwise just rely on
the default operation of php-fpm until and unless we specifically decide otherwise.

<b>ln -s STACK_ROOT/ubuntu-nginx-php-mysql/php-fpm-conf/php-fpm.conf STACK_ROOT/ubuntu-nginx-php-mysql/php/etc/php-fpm.conf</b>


3. Verify basic installation and operation:

Help, version, info.

<b>STACK_ROOT/ubuntu-nginx-php-mysql/php/sbin/php-fpm --help</b>
<b>STACK_ROOT/ubuntu-nginx-php-mysql/php/sbin/php-fpm -v</b>
<b>STACK_ROOT/ubuntu-nginx-php-mysql/php/sbin/php-fpm -i</b>

Test the configuration file:
<b>STACK_ROOT/ubuntu-nginx-php-mysql/php/sbin/php-fpm -t</b>

This will start the server.
<b>STACK_ROOT/ubuntu-nginx-php-mysql/php/sbin/php-fpm</b>

Verify that php-fpm is listening on the expected port:
<b>netstat -lnp  | grep "php-fpm"</b>
Do you see "127.0.0.1:PHP-PFM_DEFAULT_PORT" and "LISTEN"?

Restart the server and reload the config.
<b>ps -A | grep "php-fpm"</b>
<b>kill -9 nnnn nnnn nnnn ... </b>

The first command lists the processes visible to this users and will grep the results to only display those
relevant to php-fpm. 

The second command will kill these processes.  Please substitute nnnn for the actual PIDs from PS.

4. Review the directory structure, relevant to php-fpm.  Recall that php-fpm was installed with PHP so therefore
php-fpm will not have it's own original installation media or extracted source or installation directory.

<b>STACK_ROOT/ubuntu-nginx-php-mysql/php-fpm-conf</b> - This contains the versioned configuration files that we develop.

<b>STACK_ROOT/ubuntu-nginx-php-mysql/php/sbin/php-fpm</b> - This is the executable binary.

<b>STACK_ROOT/ubuntu-nginx-php-mysql/php/etc/php-fpm.conf</b> - This is a link to the configuration file.


<h3>V. Install nginx</h3>

Now it's to install nginx version 1.7.9.

<ol>
<li>Determine a port for the initial configuration to listen to.  As an unprivileged user we cannot use port 80. NGINX_DEFAULT_PORT = 3000</li>

<li>Ensure that you're in the STACK_ROOT/ubuntu-nginx-php-mysql directory.</li>

<li><b>wget http://nginx.org/download/nginx-1.7.9.tar.gz</b></li>

<li><b>tar -xvf nginx-1.7.9.tar.gz</b></li>

<li><b>cd nginx-1.7.9</b></li>

<li><b>./configure --help</b>  This is not strictly necessary, but it may be useful. Possibly review the modules being loaded and exclude some of them.</li>

<li><b>./configure --prefix=STACK_ROOT/ubuntu-nginx-php-mysql/nginx --without-http_gzip_module</b>
<br>Note: Look at the output of configure.  It will tell you that the binaries, the configuration, the logs, etc. are all found inside this directory.

<br>We don't need gzip compression right now, which requires zlib, and it's easier to omit gzip
 than to install zlib at this time.
</li>

<li><b>make --help</b>  Again, this is optional, but may be useful.</li>

</li><b>make</b></li>

<li><b>make install</b></li>

<li>Replace the installed configuration directory with the custom built configuration provided
by this project.</li>

<li><b>rm -rf STACK_ROOT/ubuntu-nginx-php-mysql/nginx/conf</b>  Remove the existing directory.</li>

<li><b>cp -R STACK_ROOT/ubuntu-nginx-php-mysql/nginx-conf/conf1 STACK_ROOT/ubuntu-nginx-php-mysql/nginx/conf</b>

The stock configuration is filled with commented out examples.  This just confuses everything.
The custom built config has _nothing_ except things we specifically want.  We'll just rely on
the default operation of nginx until and unless we specifically decide otherwise.
</li>

<li>Verify basic installation and operation:

Test the configuration file:
<b>STACK_ROOT/ubuntu-nginx-php-mysql/nginx/sbin/nginx -t</b>

This will start the server.
<b>STACK_ROOT/ubuntu-nginx-php-mysql/nginx/sbin/nginx</b>

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


