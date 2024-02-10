# Session 3

[Main Menu](../README.md) | [Session 3](../session3/README.md)

## Introduction

In [Session 2](../session2/README.md) we saw how to create an event and alarm definition using an unformatted OenNMS event from an unknown trap.
In this session we will do a worked example using the manufactures published SNMP MIBS.

![alt text](../session3/images/MotorwayCamera.jpg "Figure MotorwayCamera.jpg")

Road side cameras are used to provide surveillance for [smart motorways](https://www.highwaysmagazine.co.uk/chubb-launches-new-camera-system-for-smart-motorways/6703) and CHUBB, who manufacture these cameras, have published a MIB for camera event traps sent from their camera control system. 
In this example we will import this MIB into OpenNMS and create camera alarms.

## Getting started
In this example we will use the same network as we used in [Session 2](../session2/README.md). 

However we want to start with a clean system because instead of creating the configuration files inside the container, we will inject them using docker. 

Use the new [session3/minimal-minion-activemq](../session3/minimal-minion-activemq/) project.

```
# make sure the old database and configuration is gone by deleting the volumes using the -v option
cd minimal-minion-activemq
docker compose down -v

# restart opennms
docker compose up -d

# follow the logs until OpennMS is up.
# this will take a while because we are recreating the database
docker compose logs -f horizon

# OpenNMS will be up when you see the logs reach
horizon  | [INFO] Invoking start on object OpenNMS:Name=PerspectivePoller
horizon  | [INFO] Invocation start successful for MBean OpenNMS:Name=PerspectivePoller

```
Once OpenNMS is running, open a session at http:\\localhost:8980 (username: admin password: admin)

The provided test-network1-requisition now has two cameras

| container | Native SNMP port | Host Exposed SNMP Port | internal ip address | node label | foreign id |
| --------- | ---------------- | ---------------------- | ------------------- | ---------- | ---------- |
| chubb_camera_01 | 161        | 11561                  | 172.20.0.103        | chubb_camera_01 | chubb_camera_01 |
| chubb_camera_01 | 161        | 11661                  | 172.20.2.103        | chubb_camera_02 | chubb_camera_02 |

You will need to import the test-network1-requisition and add the `chubb` snmp community string as covered in [Exercise-2-1](../session1/Exercise-2-1.md)

You should now have the full test network including the cameras ready to design the configuration.

now test you can send traps from the cameras to OpenNMS. 

log into both cameras and try sending the following trap using netsnmp.
You should see an unformatted event in OpenNMS.

```
docker compose exec chubb_camera_01 bash

# check that snmpsim is working when you walk the MIB
 snmpwalk -v 2c -On -c chubb localhost
.1.3.6.1.2.1.1.1.0 = STRING: M1 (TUNNEL) 0/2L
.1.3.6.1.2.1.1.2.0 = OID: .1.3.6.1.4.1.52330.1.6
.1.3.6.1.2.1.1.3.0 = Timeticks: (404669) 1:07:26.69
.1.3.6.1.2.1.1.4.0 = STRING: Highways England - 03001235000
.1.3.6.1.2.1.1.5.0 = STRING: 00021,20002
.1.3.6.1.2.1.1.6.0 = STRING: 0002L

# then send a trap

snmptrap -v 2c -c public meridian:1162 ""  .1.3.6.1.4.1.52330.6.2.0.1  .1.3.6.1.4.1.52330.6.2.7.0  s xxxx   .1.3.6.1.4.1.52330.6.2.1.0 i 0  .1.3.6.1.4.1.52330.6.2.5.0 i 1
```
Do the same for chubb_camera_02

## Parsing a MIB

TBD

## todo 
exercise - worked example camera mib





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


