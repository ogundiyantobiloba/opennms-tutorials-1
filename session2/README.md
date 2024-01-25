# Session 2 OpenNMS Configuration

[Main Menu](../README.md) | [Session 2](../session2/README.md)

## Contents
1. OpenNMS Configuration Overview
* opennms configuration directories
* configuring a docker image
1. OpenNMS Events and Alarms and Traps
* basic events alarms and traps
* parsing a mib and creating an event configuration


## OpenNMS Configuration Overview

In older OpenNMS courses, we encouraged people to edit the configuration files directly in an OpenNMS system running on a virtual machine.
This worked well but it was inconvenient to have to modify the configuration for each example exercise.

In this course, it is much more convenient to provide examples using docker where the modified configuration files are simply overlaid on the default configuration files provided in the docker image. 
Please note, however that all of the examples will work equally well in a standard installation or a container installation of OpenNMS.
The configuration files being modified are exactly the same.

Before we proceed it is important to understand the main directory locations in an OpenNMS installation and how these are mapped to the example docker compose projects which use OpenNMS docker containers.
The following figure shows the folders in a typical OpenNMS installation installed on a Linux system using a package manager.
 
![alt text](../session2/images/opennmsFolders.drawio.png "Figure opennmsFolders.drawio.png")

An RPM based installation  (RHEL, Rocky Linux, Centos etc) will follow this exact pattern .

An APT based installation (Debian, Ubuntu etc)  follows the same pattern but instead of real folders, it follows the Debian directory practices of using symbolic links to the etc, logs and share directories.

All of the configuration files are held in the `/opt/opennms/etc/` directory.

The `/opt/opennms/share/etc-pristine` directory always holds the untouched original configuration files shipped with any particular OpenNMS distribution. 
This always allows you to compare any changes with the default state.

In a non-containerised install, we advise users to version control the `/opt/opennms/etc/` directory so that you can track all changes made locally. 
A simple but effective approach to this is to turn the `/opt/opennms/etc/` directory into a git repository and base line on the untouched files. 

In containerised installs, you should version control the configurations injected into the container.

The `/opt/opennms/share/xsds` directory contains the XML Schema Definitions for the xml files. 
These xsd definitions are generated from the jaxb annotated code during the build of OpenNMS and provide detailed documentation for all of the fields in the xml configuration files.

---
**NOTE**

For these tutorials, we have provided a folder of untouched xml configurations and associated xsds in [pristine-opennms-config-files](../../main/pristine-opennms-config-files/)

You can copy and modify these as you need to in the docker compose configuration overlays.

---

## docker container mapping

OpenNMS Docker containers follow a similar pattern to the standard Linux installations, with earlier containers based on Centos and later containers using Ubuntu as the based image. 

However it is not enough just to overwrite the default container directories as the containers have been designed for easy configuration using environment variables and you need to follow additional conventions when injecting configuration files into an OpenNMS container.

### environment variables and confd

OpenNMS containers use [confd](https://github.com/kelseyhightower/confd/tree/master) templates to create some etc configurations on startup based on injected environment variables and templates. 

The following environment variables can be injected to the container, but if they are not specified in the docker-compose.yml scripts, the shown defaults are used.
(This is why the variables are not always specified in the examples).

```
    environment:
      TZ: 'America/New_York'(4)
      POSTGRES_HOST: 'database'(5)
      POSTGRES_PORT: 5432
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      OPENNMS_DBNAME: 'opennms'
      OPENNMS_DBUSER: 'opennms'
      OPENNMS_DBPASS: 'opennms'
```
These particular database settings set values in the file [/etc/opennms-datasources.xml](../../main/pristine-opennms-config-files/etc-pristine/opennms-datasources.xml)

If you overlay `opennms-datasources.xml`, the environment variables will not be applied on container start.

The minion containers use an injected configuration file [/opt/minion/minion-config.yaml](../session2/minimal-minion-activemq/container-fs/minion1/opt/minion/minion-config.yaml) to set up the internal minion /etc/*.cfg properties.

### overlay files 

Secondly, on startup, OpenNMS core containers copy any files in `/opt/opennms-overlay/` and replace the default files in `/opt/opennms/etc/`

So in all of the examples, the relevant files are modified in the docker compose project and injected into the container `/opt/opennms-overlay/` so that they will be copied to the `/opt/opennms/etc` folder before OpenNMS starts.

We needed to cover this introduction to configuration so that you understand how the examples relate to your production OpenNMS installation. 
We will have more to say about configuration as we proceed with the course.

## Provisioning Requisitions

TBC
Provisioning  requisitions
netsim container
SNMP Community strings

exercise 1
note docker compose down -v first


## Events, Alarms and Traps

TBC

Node down node up events and traps
Node down node up alarms

Exercise - happiness alarm
happines alarm
event from trap
event from UI
event from ReST / Rester

Exercise - parsing a mib for events and creating alarms - chubb mib

```
[root@netsnmp_1_1 /]# snmpwalk -v 2c -On -c chubb chubb_camera_01
.1.3.6.1.2.1.1.1.0 = STRING: M1 (TUNNEL) 0/2L
.1.3.6.1.2.1.1.2.0 = OID: .1.3.6.1.4.1.52330.1.6
.1.3.6.1.2.1.1.3.0 = Timeticks: (404669) 1:07:26.69
.1.3.6.1.2.1.1.4.0 = STRING: Highways England - 03001235000
.1.3.6.1.2.1.1.5.0 = STRING: 00021,20002
.1.3.6.1.2.1.1.6.0 = STRING: 0002L
[root@netsnmp_1_1 /]# snmpwalk -v 2c -On -c public chubb_camera_01
```
RUN echo "deb http://archive.debian.org/debian stretch main" > /etc/apt/sources.list

