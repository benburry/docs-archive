---
layout: default
title: "PE 2015.2 » Installing » Components, Logs, and License Files"
subtitle: "Components, Logs, and License Files: What Gets Installed and Where?"
canonical: "/pe/latest/install_what_and_where.html"
---



## License File

Your PE license file (which was emailed to you when you purchased Puppet Enterprise) should be placed in `/etc/puppetlabs/license.key`.

Puppet Enterprise can be evaluated with a complimentary ten-node license; beyond that, a commercial per-node license is required for use. A license key file is emailed to you after your purchase. On a monolithic installation, the Puppet master looks for this key at `/etc/puppetlabs/license.key`. On a split installation, it looks for this key at `/etc/puppetlabs/license.key` on the Puppet master and the PE console nodes. Puppet will log warnings if the license is expired or exceeded. You can view the status of your license by running `puppet license` at the command line on the Puppet master.

To purchase a license, please see the [Puppet Enterprise pricing page](http://www.puppetlabs.com/puppet/how-to-buy/), or contact Puppet Labs at <sales@puppetlabs.com> or (877) 575-9775. For more information on licensing terms, please see [the licensing FAQ](http://www.puppetlabs.com/licensing-faq/). If you have misplaced or never received your license key, please contact <sales@puppetlabs.com>.

### License Files in the Console

The **Licenses** menu shows you the number of nodes that are currently active and the number of nodes still available on your current license. If the number of available licenses is exceeded, a warning will be displayed. The number of licenses used is determined by the number of active nodes known to PuppetDB. This is a change from previous behavior which used the number of unrevoked certs known by the CA to determine used licenses. The menu item provides convenient links to purchase and pricing information.

Unused nodes will be deactivated automatically after seven days with no activity (no new facts, catalog or reports), or you can use `puppet node deactivate` for immediate results.

## Puppet Enterprise Software Components

### What

The functional components of PE, are separated between those packaged with the all-in-one (AIO) puppet-agent and those packaged on the server side (which includes the AIO puppet-agent). PE 2015.2 includes the following major software components.

{% partial /pe/_versions_2015.md %}

### Where

On \*nix nodes, all PE software (excluding config files and generated data) is installed under `/opt/puppetlabs`.

On Windows nodes, all PE software is installed in the "Puppet Enterprise" subdirectory of [the standard 32-bit applications directory](./install_windows.html#program-directory)

* Executable binaries on \*nix are in `/opt/puppetlabs/bin` and `/opt/puppetlabs/sbin`. The Installation Overview provides more info about [PE binaries and symlinks](./install_basic.html#puppet-enterprise-binaries-and-symlinks)
* The Puppet modules included with PE are installed on the Puppet master server in `/opt/puppetlabs/puppet/modules`. Don't modify anything in this directory or add modules of your own. Instead, install them in `/etc/puppetlabs/code/environments/<environment>/modules`.
* Orchestration plugins are installed in `/opt/puppetlabs/mcollective/plugins/` on \*nix and in [`<COMMON_APPDATA>`](./install_windows.html#data-directory)`\PuppetLabs\mcollective\etc\plugins\mcollective` on Windows. If you are adding new plugins to your PE agent nodes, you should [distribute them via Puppet as described in the "Adding Actions" page of this manual](./orchestration_adding_actions.html).

### Dependencies

For information about PostgreSQL and OpenSSL requirements, refer to the [system requirements](./install_system_requirements.html#dependencies-and-os-specific-details).

### Configuration Files

#### What

Most files used to configure Puppet Enterprise and its subsidiary components are managed by Puppet. The files listed below  are files you may need to change to accommodate the needs of your environment.

#### Where

On *nix nodes, Puppet Enterprise's configuration files all live under `/etc/puppetlabs/puppet`.

On Windows nodes, Puppet Enterprise's configuration files all live under `<COMMON_APPDATA>\PuppetLabs`. The location of this folder [varies by Windows version](./install_windows.html#data-directory); in 2008 and 2012, its default location is `C:\ProgramData\PuppetLabs\puppet\etc`.

PE's various components all have subdirectories inside this main data directory:

* Puppet's `confdir` is in the `puppet` subdirectory. This directory contains the [`puppet.conf`](/guides/configuring.html) file, `auth.conf`, and the SSL directory.

### Log Files

The software distributed with Puppet Enterprise generates the following log files, which can be found as follows.

#### Puppet Master Logs

The Puppet master service's logging are contained in two files:

* `/var/log/puppetlabs/puppetserver/puppetserver.log`: the Puppet master application logs its activity here; this is where things like compilation errors and deprecation warnings can be found.
* `/var/log/puppetlabs/puppetserver/puppetserver-daemon.log`: this is where fatal errors or crash reports can be found.

#### Puppet Agent Logs

On *nix nodes, the Puppet agent service logs its activity to the syslog service. Your syslog configuration dictates where these messages will be saved, but the default location is `/var/log/messages` on Linux, `/var/log/system.log` on Mac OS X, and `/var/adm/messages` on Solaris.

On Windows nodes, the Puppet agent service logs its activity to the Windows Event Log. You can view its logs by browsing the Event Viewer. (Control Panel → System and Security → Administrative Tools → Event Viewer)

#### ActiveMQ Logs

- `/var/log/puppetlabs/activemq/wrapper.log`
- `/var/log/puppetlabs/activemq/activemq.log`
- `/var/log/puppetlabs/activemq/data/audit.log`

#### Orchestration Service Log

- `/var/log/puppetlabs/mcollective.log`: maintained by the orchestration service, which is installed on all nodes.
- `/var/log/puppetlabs/mcollective-audit.log`: exists on all nodes that have mcollective installed; logs any mcollective actions run on the node, including information about the client that called the node

#### Console Logs

- `/var/log/puppetlabs/nginx/error.log`: contains errors related to nginx. Console errors that don't get logged anywhere else can be found in this log. If you have problems with the console or Puppet, this log may be useful.
- `/var/log/puppetlabs/console-services.log`
- `/var/log/puppetlabs/console-services-access.log`
- `/var/log/puppetlabs/console-services-daemon.log`: this is where fatal errors or crash reports can be found.

#### Installer Logs

- `/var/log/puppetlabs/installer/http.log`: contains the web requests sent to the installer; present only on the machine from which the web-based install was performed
- `/var/log/puppetlabs/installer/installer-<timestamp>.log`: contains the operations performed and any errors that occurred during installation
- `/var/log/puppetlabs/installer/answers.install`: contains the contents of the answer file used to install PE; passwords are redacted from this file
- `/var/log/puppetlabs/installer/install_log.lastrun.<hostname>.log`: contains the contents of the last installer run

#### Database Log

- `/var/log/puppetlabs/puppetdb/puppetdb.log`
- `/var/log/puppetlabs/postgresql/pgstartup.log`

#### r10k Logs


### Tools for Working with Puppet Enterprise

PE installs several suites of tools to help you work with the major components of the software. These include:

- **Puppet Tools** --- Tools that control basic functions of Puppet such as `puppet master,` `puppet apply` and `puppet cert.`
    See the [Tools section](/guides/tools.html) of the Puppet Manual for more information.
- **Orchestration Tools** --- Tools used to orchestrate simultaneous actions across a number of nodes. These tools are built on the MCollective framework and are accessed via the `mco` command. See the [Orchestration documentation](./orchestration_overview.html) for more information.
- **Module Tools** --- The Module tool is used to access and create Puppet Modules, which are reusable chunks of Puppet code users have written to automate configuration and deployment tasks. For more information, and to access modules, visit the [Puppet Forge](http://forge.puppetlabs.com/).
- **Console** --- The console is Puppet Enterprise's web UI. The console provides tools to view and edit resources on your nodes, view reports and activity graphs, etc. See the [Console section](./console_accessing.html) of the Puppet Manual for more information.

For more details, you can also refer to the man page for a given command or subcommand.

### Services

PE uses the following services:

- **`pe-activemq`** --- The ActiveMQ message server, which passes messages to the MCollective servers on agent nodes. Runs on servers with the Puppet master component.
- **`pe-console-services`** --- Manages and serves the PE console.
- **`pe-puppetserver`** --- The Puppet master server, which manages the Puppet master component.
- **`pe-nginx`** --- Nginx, serves as a reverse-proxy to the PE console.
- **`mcollective`** --- The orchestration (MCollective) daemon, which listens for orchestration messages and invokes actions. Runs on every agent node.
- **`puppet`** (on EL and Debian-based platforms) --- The Puppet agent daemon. Runs on every agent node.
- **`pe-puppetdb`** and **`pe-postgresql`** --- Daemons that manage and serve the database components. Note that pe-postgresql is only created if we install and manage PostgreSQL for you.

### User Accounts

PE creates the following users:

- **`peadmin`** --- An administrative account which can invoke orchestration actions. This is the only PE user account intended for use in a login shell. See [the "Invoking Orchestration Actions" page of this manual](./orchestration_invoke_cli.html) for more about this user. This user exists on servers with the Puppet master component.
- **`pe-puppet`**  --- A system user that runs the Puppet master processes spawned by pe-puppetserver.
- **`pe-webserver`** --- A system user that runs Nginx (`pe-nginx`).
- **`pe-activemq`** --- A system user that runs the ActiveMQ message bus used by MCollective.
- **`pe-puppetdb`** --- A system user with root access to the database.
- **`pe-postgres`** --- A system user with access to the pe-postgreSQL instance. Note that this user is only created if we install and manage PostgreSQL for you.
- **`pe-console-services`** --- A system user that runs the console process.

### Group Accounts

PE creates the following groups:

- **`peadmin`** --- An administrative group which can invoke orchestration actions.
- **`pe-puppet`** --- A system group that runs the Puppet master processes spawned by pe-puppetserver.
- **`pe-webserver`** --- A system group that runs Nginx (`pe-nginx`).
- **`pe-activemq`** --- A system group that runs the ActiveMQ message bus used by MCollective.
- **`pe-puppetdb`** --- A system group with root access to the database.
- **`pe-postgres`** --- A system group with access to the pe-postgreSQL instance. Note that this group is only created if we install and manage PostgreSQL for you.
- **`pe-console-services`** --- A system group that runs the console process.

### Certificates

During install, PE generates the following certificates (can be found at `/etc/puppetlabs/puppet/ssl/certs`):

- **`<user-entered console certname>`** ---  The certificate for the PE console. Only generated if the user has chosen to install the console in a split component configuration.
- **`<user entered PuppetDB certname>`** ---  The certificate for the database component. Only generated if the user has chosen to install the database in a split component configuration.
- **` <user-entered master certname> `** --- This certificate is either generated at install if the Puppet master and console are the same machine or is signed by the master if the console is on a separate machine.
- **`pe-internal-mcollective-servers`** --- A shared certificate generated on the Puppet master and shared to all agent nodes.
- **`pe-internal-peadmin-mcollective-client`** --- The orchestration certificate for the peadmin account on the Puppet master.
- **`pe-internal-puppet-console-mcollective-client`** --- The orchestration certificate for the PE console.
- **`pe-internal-classifier`** --- The certificate for the node classifier.
- **`pe-internal-dashboard`** --- The certificate for the PE console.

A fresh PE install should thus give the following list of certificates:

    root@master:~# puppet cert list --all
    + "master"                                        (40:D5:40:FA:E2:94:36:4D:C4:8C:CE:68:FB:77:73:AB) (alt names: "DNS:master", "DNS:puppet", "DNS:puppet.soupkitchen.internal")
    + "pe-internal-classifier"                        F3:57:28:CC:72:2A:8B:75:93:65:27:99:AC:72:B4:AF
    + "pe-internal-dashboard"                         F6:07:C7:E9:33:76:D4:3E:12:D7:77:6D:85:0B:2A:D1
    + "pe-internal-mcollective-servers"               C2:59:69:2E:A1:45:7D:BA:BC:63:71:74:83:03:BF:49
    + "pe-internal-peadmin-mcollective-client"        85:56:72:72:04:02:8E:61:8C:34:B8:53:34:0A:3E:8C
    + "pe-internal-puppet-console-mcollective-client" A1:16:FB:12:9B:3E:85:2A:BF:28:C7:46:A9:8C:E7:55

### Documentation

Man pages for the Puppet subcommands are generated on the fly. To view them, run `puppet man <SUBCOMMAND>`.

The `pe-man` command from previous versions of Puppet Enterprise is no longer functional. Use the above method instead.



* * *

- [Next: Accessing the Console](./console_accessing.html)
