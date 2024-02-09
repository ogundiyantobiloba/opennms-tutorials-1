# Session 3

[Main Menu](../README.md) | [Session 3](../session3/README.md)


TBD

## todo 

Event Translator
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


