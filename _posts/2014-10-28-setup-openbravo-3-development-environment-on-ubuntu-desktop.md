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

This is a guide on how to install Openbravo 3.0 and required software components, to get Openbravo Development Environment up and running on Ubuntu Desktop.

### 1. PostgreSQL 9.1

Current version is 9.3, but Openbravo recommends 9.1. Check and remove existing versions if necessary:  

    $ psql --version
    $ dpkg --get-selections | grep postgres
    $ sudo apt-get remove postgresql-*
    $ sudo apt-get purge postgresql-*
    $ dpkg --get-selections | grep postgres

Create file `/etc/apt/sources.list.d/pgdg.list` and add the following line:

> deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main

Install PostgreSQL 9.1, including contrib and pgadmin3 packages, and check if it's running:

    $ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
    $ sudo apt-get update && sudo apt-get upgrade
    $ sudo apt-get install postgresql-9.1 postgresql-contrib-9.1 pgadmin3
    $ sudo service postgresql status

To allow remote connections, add following line to the `/etc/postgresql/9.1/main/postgresql.conf`:

> listen_addresses='*'

To accept password auth, add following line to the `/etc/postgresql/9.1/main/pg_hba.conf`:

> host	all	all	0.0.0.0/0	md5

Set "postgres" user password and restart PostgreSQL server):

    $ sudo -u postgres psql postgres
    # \password postgres
    # \q
    $ sudo service postgresql restart

Check remote connection to PostgreSQL Server.

### 2. Sun JDK 1.6 (1.6 update 45)

Current version is 1.8.0_25, but Openbravo recommends latest 1.6 update (1.6.0_45). Download `jdk-6u45-linux-x64.bin` from Oracle's site. Copy installation file to appropriate location (eg. `~/java/jdk`), set executable and install:

    $ chmod +x jdk-6u45-linux-x64.bin
    $ ./jdk-6u45-linux-x64.bin
    $ rm ~/java/jdk/jdk-6u45-linux-x64.bin

If you followed this method, JDK should be installed under `~/java/jdk/jdk1.6.0_45` directory.

### 3. Apache Ant 1.8.4

Current version is 1.9.4 but Openbravo recommends latest 1.8.x (1.8.4). Download `apache-ant-1.9.4-bin.zip`. Copy zip file to appropriate location (eg. `~/java/tools`) and extract it:

    $ unzip apache-ant-1.8.4-bin.zip
    $ rm apache-ant-1.8.4-bin.zip

If you followed this method, Ant should be installed under `~/java/tools/apache-ant-1.8.4` directory.

### 4. Apache Tomcat 6.0.41

Current version is 8.0.14 but Openbravo recommends latest 6.0.x (6.0.41). Download `apache-tomcat-6.0.41.zip`. Copy install file to appropriate location (eg. `~/java/app-servers`) and extract it:

    $ unzip apache-tomcat-6.0.41.zip
    $ rm apache-tomcat-6.0.41.zip 

If you followed this method, Ant should be installed under `~/java/app-servers/apache-tomcat-6.0.41` directory.

### 5. Environment variables

Add following variables to your `~/.profile` file:

> export JAVA_HOME=/home/\<user\>/java/jdk/jdk1.6.0_45<BR/>
> export ANT_HOME=/home/\<user\>/java/tools/apache-ant-1.8.4<BR/>
> export ANT_OPTS="-Xmx1024M -XX:MaxPermSize=128M"<BR/>
> export PATH=$PATH:$JAVA_HOME/bin:$ANT_HOME/bin<BR/>

Load variables into SSH session, or restart Ubuntu server:

    $ source .profile

### 6. Openbravo 3.0PR14Q3.2

Current version is 3.0PR14Q3.2. Download `openbravo-3.0PR14Q3.2.tar.bz2`. Copy installation file to appropriate location (eg. `~/java/workspace/ob3/<project-name>`), extract it and rename:

    $ tar xvjf openbravo-3.0PR14Q3.2.tar.bz2
    $ mv Openbravo-3.0PR14Q3.2/ openbravo
    $ rm openbravo-3.0PR14Q3.2.tar.bz2

Download setup-properties via ant:

    $ cd ~/java/workspace/ob3/<project-name>/openbravo 
    $ ant setup -autoproxy

If there's a problem with proxy, try downloading it manually:

    $ cd ~/java/workspace/ob3/<project-name>/openbravo/config
    $ wget https://code.openbravo.com/tools/rm/erp-setup-tool/raw-file/tip/setup/output/setup-properties-linux-x64.bin

Execute and setup properties

    $ chmod +x setup-properties-linux-x64.bin 
    $ ./setup-properties-linux-x64.bin

Properties:

- date separator: `.`
- attachments folder: `.../<project-name>/attachments`
- DB name: `openbravo_<project-name>`
- other appropriate db settings (host, port, user, pass)
- `deploy.mode`: `none` (edit Openbravo.properties directly)
- `safe.mode`: `false` (edit Openbravo.properties directly)

Remove sample reference data if you don't need it.

Run install source and track output:

    $ ant install.source > install.source.out

After the task has completed the log should not contain any error or exception massages as well as it should have BUILD SUCCESSFUL message at the end of the file.

    $ grep error install.source.out

### 7. Eclipse 4.4.0 (EE)

Current version is Java EE 4.4.0 (Luna), 64 bit.

Preparation:

- select workspace: `~/java/workspace/ob3/<project-name>`
- disable `Project->Build Automatically`
- set `Window->Preferences->Java->Compiler->Compiler Compliance Level` to 1.6
- set `Window->Preferences->Java->Installed JREs`: add previously installed `jdk1.6.0_45` and mark as default
- restart workspace

Complete details about importing and appropriate screenshots can be found [here](http://wiki.openbravo.com/wiki/How_to_setup_Eclipse_IDE#Import_into_Eclipse_IDE).

Summary of steps:

- import `openbravo`, `src-core`, `src-trl`, `src-wad` projects (without copying to workspace)
- add tomcat server to eclipse (use installation added in step 4. of this guide) and add openbravo as deployed resource
- setup tomcat using [this guide](http://wiki.openbravo.com/wiki/How_to_setup_Eclipse_IDE#Change_VM_arguments_settings)
- import Eclipse preferences
- `Refresh all` and `Build all` (must not be any error)
- start tomcat and check if openbravo web application is working

### Appendix: References

- System requirements: http://wiki.openbravo.com/wiki/System_Requirements
- Development stack setup: http://wiki.openbravo.com/wiki/Development_Stack_Setup
- How to setup Eclipse IDE: http://wiki.openbravo.com/wiki/How_to_setup_Eclipse_IDE
- Installing older PostgreSQL version: http://ubuntuhandbook.org/index.php/2014/02/install-postgresql-ubuntu-14-04/

