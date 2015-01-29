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


