---
layout: post
title: "Setup Openbravo 3 Development Environment on Ubuntu Desktop"
modified:
categories: 
excerpt: "Guide on how to install Openbravo 3.0 and required software components, to get Openbravo Development Environment up and running on Ubuntu Desktop."
tags: [openbravo, development environment, java, eclipse, ant, tomcat, ubuntu, postgresql]
comments: true
image:
  feature:
date: 2014-10-28T14:38:05+01:00
---

This post will guide you through installing Openbravo 3.0 and required software components, to get Openbravo Development Environment up and running on Ubuntu Desktop.

### 1. PostgreSQL 9.1

[Openbravo recommends](http://wiki.openbravo.com/wiki/System_Requirements#Server:_software_stack) latest PostgreSQL 9.1.x version. Check already installed postgres packages:

    $ dpkg --get-selections | grep postgres

If you already have `postgresql-9.1` and `postgresql-contrib-9.1` packages installed, skip the rest of this section. Otherwise, create a file named `/etc/apt/sources.list.d/pgdg.list` and add the following line:

> deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main

Now install PostgreSQL 9.1, including contrib and pgadmin3 packages, and check if `postgresql` service is running:

    $ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
    $ sudo apt-get update && sudo apt-get upgrade
    $ sudo apt-get install postgresql-9.1 postgresql-contrib-9.1 pgadmin3
    $ sudo service postgresql status

To allow remote connections to PostgreSQL server, add the following line to the config file `/etc/postgresql/9.1/main/postgresql.conf`:

> listen_addresses=\'\*\'

, and following one to another config file `/etc/postgresql/9.1/main/pg_hba.conf`:

> host	all	all	0.0.0.0/0	md5

Set password for `postgres` user and restart PostgreSQL server for all changes to take effect:

    $ sudo -u postgres psql postgres
    # \password postgres
    # \q
    $ sudo service postgresql restart

Check remote connection to PostgreSQL Server.

### 2. Oracle JDK 1.6 (1.6 update 45)

[Openbravo recommends](http://wiki.openbravo.com/wiki/System_Requirements#Server:_software_stack) latest Oracle JDK 1.6.x version (1.6.0_45). Download `jdk-6u45-linux-x64.bin` from Oracle's site. Copy installation file to appropriate location (eg. `~/java/jdk`), set executable and install:

    $ chmod +x jdk-6u45-linux-x64.bin
    $ ./jdk-6u45-linux-x64.bin
    $ rm ~/java/jdk/jdk-6u45-linux-x64.bin

If you have followed this method, JDK should be installed under `~/java/jdk/jdk1.6.0_45` directory.

### 3. Apache Ant 1.8.4

[Openbravo recommends](http://wiki.openbravo.com/wiki/System_Requirements#Server:_software_stack) latest Apache Ant 1.8.x version (1.8.4). Download `apache-ant-1.8.4-bin.zip`. Copy zip file to appropriate location (eg. `~/java/tools`) and extract it:

    $ unzip apache-ant-1.8.4-bin.zip
    $ rm apache-ant-1.8.4-bin.zip

If you have followed this method, Ant should be installed under `~/java/tools/apache-ant-1.8.4` directory.

### 4. Apache Tomcat 6.0.41

[Openbravo recommends](http://wiki.openbravo.com/wiki/System_Requirements#Server:_software_stack) latest Apache Tomcat 6.0.x version (6.0.41). Download `apache-tomcat-6.0.41.zip`. Copy zip file to appropriate location (eg. `~/java/app-servers`) and extract it:

    $ unzip apache-tomcat-6.0.41.zip
    $ rm apache-tomcat-6.0.41.zip 

If you have followed this method, Tomcat should be installed under `~/java/app-servers/apache-tomcat-6.0.41` directory.

### 5. Environment variables

Add following lines (variables) to your `~/.profile` file:

> export JAVA_HOME=/home/\<user\>/java/jdk/jdk1.6.0_45<BR/>
> export ANT_HOME=/home/\<user\>/java/tools/apache-ant-1.8.4<BR/>
> export ANT_OPTS=\"-Xmx1024M -XX:MaxPermSize=128M\"<BR/>
> export PATH=$PATH:$JAVA_HOME/bin:$ANT_HOME/bin<BR/>

Load variables into SSH session (or restart Ubuntu):

    $ source .profile

### 6. Openbravo 3.0PR14Q3.2

Current version of Openbravo is 3.0PR14Q3.2. Download `openbravo-3.0PR14Q3.2.tar.bz2`. Copy installation file to appropriate location (eg. `~/java/workspace/ob3/<project-name>`), extract it and rename root folder:

    $ tar xvjf openbravo-3.0PR14Q3.2.tar.bz2
    $ mv Openbravo-3.0PR14Q3.2/ openbravo
    $ rm openbravo-3.0PR14Q3.2.tar.bz2

Download setup properties program using Ant (use `-autoproxy` argument if you are behind the proxy):

    $ cd ~/java/workspace/ob3/<project-name>/openbravo 
    $ ant setup -autoproxy

If there's a problem with establishing connection to servers, try downloading file manually:

    $ cd ~/java/workspace/ob3/<project-name>/openbravo/config
    $ wget https://code.openbravo.com/tools/rm/erp-setup-tool/raw-file/tip/setup/output/setup-properties-linux-x64.bin

Make program executable and set necessary  properties:

    $ chmod +x setup-properties-linux-x64.bin 
    $ ./setup-properties-linux-x64.bin

Properties to set:

- date separator: `.`
- attachments folder: `.../<project-name>/attachments`
- DB name: `openbravo_<project-name>`
- other db settings (host, port, user, pass)
- `deploy.mode`: `none` (edit Openbravo.properties directly)
- `safe.mode`: `false` (edit Openbravo.properties directly)

Remove sample reference data if you don't need it. Run install source and track output:

    $ ant install.source > install.source.out

After the task has completed the log should not contain any error or exception massages as well as it should have BUILD SUCCESSFUL message at the end of the file.

    $ grep error install.source.out

### 7. Eclipse 4.4.0 (EE)

Current Eclipse version is Luna 4.4.1. Prepare your IDE:

- select workspace: `~/java/workspace/ob3/<project-name>`
- disable `Project -> Build Automatically`
- set `Window -> Preferences -> Java -> Compiler -> Compiler Compliance Level` to `1.6`
- go to `Window -> Preferences -> Java -> Installed JREs` and add previously installed `jdk1.6.0_45` and mark as default
- restart workspace

Complete details about importing Openbravo projects into Eclipse, with appropriate screenshots, can be found [here](http://wiki.openbravo.com/wiki/How_to_setup_Eclipse_IDE#Import_into_Eclipse_IDE). Short summary of steps:

- import `openbravo`, `src-core`, `src-trl`, `src-wad` projects (without copying to workspace)
- add tomcat server to eclipse (use Tomcat installation added in step 4. of this guide) and add openbravo as deployed resource
- set necessary tomcat properties using [this guide](http://wiki.openbravo.com/wiki/How_to_setup_Eclipse_IDE#Change_VM_arguments_settings)
- import Eclipse preferences
- `Refresh all` and `Build all` projects within Eclipse (there must not be any error)
- start tomcat and check if openbravo web application is working

### Appendix: References

- [System requirements](http://wiki.openbravo.com/wiki/System_Requirements)
- [Development stack setup](http://wiki.openbravo.com/wiki/Development_Stack_Setup)
- [How to setup Eclipse IDE](http://wiki.openbravo.com/wiki/How_to_setup_Eclipse_IDE)
- [Installing older PostgreSQL version](http://ubuntuhandbook.org/index.php/2014/02/install-postgresql-ubuntu-14-04/)

