# Exercise 2-2 

[Main Menu](../README.md) | [Session 2](../session2/README.md) | [Exercise-2-2](../session1/Exercise-2-2.md)

# OpenNMS Events

## browsing the mibs using MibBrowser

A very useful free tool for browsing SNMP MIBS and  generating SNMP traps is the Ireasoning Mib Browser which can be installed on Windows or Linux and is downloaded from 

[https://www.ireasoning.com/mibbrowser.shtml](https://www.ireasoning.com/mibbrowser.shtml)

Please install this software on your host system so we can use it to interrogate the SNMP agents in the example containers.

By convention,  SNMP agents use port 161 for responding to SNMP requests and 162 for receiving traps.

However, in order to avoid conflicts with Linux NetSNMP, OpenNMS is usually set up to use a different port to receive traps.
The OpenNMS core SNMP trap port is set in the file [etc/trapd-configuration.xml](../../main/pristine-opennms-config-files/etc-pristine/trapd-configuration.xml)

It is worth noting that this file also contains a setting `new-suspect-on-trap="false"` which if set `true` will cause OpenNMS to scan for a new node if it does not recognise the from IP address of the trap.

In our example [docker-compose.yaml](../session2/minimal-minion-activemq/docker-compose.yaml) file, you will see that each of the netsnmp containers exposes a different port on the host system. 
And the core OpenNMS horizon and the minion1 receive SNMP traps from the host on 10162 and 1162 respectively.

| container | Native SNMP port | Host Exposed SNMP Port |
| --------- | ---------------- | ---------------------- |
| netsnmp_1_1 | 161 | 11161 |
| netsnmp_1_2 | 161 | 11261 |
| netsnmp_2_1 | 161 | 11361 |
| netsnmp_2_2 | 161 | 11461 |
| chubb_camera_01 | 161 | 11561 |
| horizon | 1162 | 10162 |
| minion1 | 1162 | 1162 |

We can try walking the mib of netsnmp_1_1 using the mibbrowser.

Set the address to localhost and open the advanced tab to set the port to `11161` and the community strings to `public`.

![alt text](../session2/images/mibbrowser-advanced.png "Figure mibbrowser-advanced.png")

When you select operations > walk and then GO, you should see output like this image.

![alt text](../session2/images/mibbrowser-walk1.png "Figure mibbrowser-walk1.png")

Note the line `Name/OID: sysObjectID.0; Value (OID): .1.3.6.1.4.1.8072.3.2.10`

The sysObjectID will be different for every vendor and device and is assigned by IANA.
You can search a list of susObjectID mappings per vendor here [https://www.iana.org/assignments/enterprise-numbers/](https://www.iana.org/assignments/enterprise-numbers/)

The sysObjectID is the primary means by which OpenNMS knows what sort of device it is and therefore what MIBS are available to collect data from.

`8072` is the registered sysObjectID for NetSNMP. 

`5813` is the registered sysObjectID for The OpenNMS Group, Inc.  (this is often used when OpenNMS forwards traps to other systems)

Try walking the other containers by using a different port.
Note that you will need to use community string `chubb` for chubb_camera_01

## generating an event using MibBrowser

You can use the MibBrowser to generate traps which are sent to OpenNMS.

Select Tools > trap sender

The figure below shows sending a LinkDown trap to horizon on port 1162

With an SNMP V1 Trap you can specify the source IP address and in this case we have specified that the trap is coming from  `172.20.0.101` which corresponds to netsnmp_1_1.

---
**NOTE**

SNMP v1 allows the source IP address to be set as a varbind but  with SNMP v2 traps, you can't specify the source IP as a varbind.
In this case the UDP message must come from the actual device IP address set by the operating system.
See an explanation here https://stackoverflow.com/questions/76741423/pysnmp-impossible-to-change-source-address

---

![alt text](../session2/images/mibbrowser-sendlinkdowntrap.png "Figure mibbrowser-sendlinkdowntrap.png")

Send a down tap several times and then look at the OpenNMS event list to see that the traps have arrived as Agent Interface Down (linkDown Trap) events with WARNING severity.

http://localhost:8980/opennms/event/list

![alt text](../session2/images/onms-eventlist1.png "Figure onms-eventlist1.png")

On a separate tab, open the OpenNMS alarm list and you should see a single WARNING Alarm Agent Interface Down (linkDown Trap)  with a count of events.

![alt text](../session2/images/onms-alarmlist1.png "Figure onms-alarmlist1.png")

Now send a LinkUp trap from `172.20.0.101`

You should see a NORMAL severity link up event in the event list

You should also see that the corresponding alarm in the alarm list is now CLEARED and after a few minutes, it will be deleted from the list.

## generating an unknown trap using mibbrowser

![alt text](../session2/images/sendTrap1.png "Figure sendTrap1.png")


## generating events using netsnmp

MibBrowser is very useful but we actually want to simulate v2 traps coming from devices within our docker network.
In this case we will use NetSNMP to send traps from the command line. 
The following gives some basic examples but you should see the man page or the [NetSNMP documentation](http://www.net-snmp.org/) for more details.

To send SNMP v2 taps, the  command format is 

snmptrap [OPTIONS] AGENT TRAP-PARAMETERS

The trap parameters are also called `varbinds` and consist of oid/type/value  where the type of the data can vary but we are using 'i' integer or 's ' string (for other possible types see [NetSNMP snmpset documentation](http://www.net-snmp.org/wiki/index.php/TUT:snmpset) i: INTEGER, u: unsigned INTEGER, t: TIMETICKS, a: IPADDRESS, o: OBJID, s: STRING, x: HEX STRING, d: DECIMAL STRING).

Although officially, the OID of each varbind must be defined in the trap, OpenNMS is not concerned with and doesn't use the varbind OID but only the POSITION of the varbind value. (i.e. is it the first, second or third value etc. in the sequence). 
So in these examples, Provided a varbind OID is set, it doesn't matter what the OID is as it is only the order of the varbinds which are important to OpenNMS.
(Please note however that the event translator may care about the varbind OID but more on this later)

The following breaks down the content of a trap to be sent using snmptrap:

```
snmptrap -v 2c         \ # sets the trap version to v2c
         -c public     \ # sets the community string to public
         meridian:1162 \ # host:port. If port is omitted, 162 will be used
         ""            \ # supplying no value by using two single quotes '' uses the operating system up time. Alternatively the format is  36:2:40:51.67 which equates to 36 days, 2 hours, 40 minutes and 51.67 seconds
          .1.3.6.1.4.1.52330.6.2.0.1 \ #  trapoid
           .1.3.6.1.4.1.52330.6.2.7.0  s xxxx  \ # sequence of OID TYPE VALUE Here s is a string value xxxx
           .1.3.6.1.4.1.52330.6.2.1.0  i 0  \ # here i is an integer of value 0 
           .1.3.6.1.4.1.52330.6.2.5.0  i 1    # here i is an integer of value 1
```

Putting this all together we get 

```
snmptrap -v 2c -c public meridian:1162 ""  .1.3.6.1.4.1.52330.6.2.0.1  .1.3.6.1.4.1.52330.6.2.7.0  s xxxx   .1.3.6.1.4.1.52330.6.2.1.0 i 0  .1.3.6.1.4.1.52330.6.2.5.0 i 1
```
This sends an SNMP v2c trap of type .1.3.6.1.4.1.52330.6.2.0.1 with the first varbind as a string, the second as an integer 0 and the third as an integer 1

Try sending this trap to opennms horizon from the netsnmp_1_1 container


```
# log into the netsnmp_1_1 container

docker compose exec netsnmp_1_1 bash

# ping horizon to ensure you can reach it from the container
[root@netsnmp_1_1 /]# ping horizon
PING horizon (172.20.0.15) 56(84) bytes of data.
64 bytes from horizon.minimal-minion-activemq_N000 (172.20.0.15): icmp_seq=1 ttl=64 time=1.48 ms

# send a trap to meridian port 1162 using netsnmp

snmptrap -v 2c -c public horizon:1162 ""  .1.3.6.1.4.1.52330.6.2.0.1        .1.3.6.1.4.1.52330.6.2.7.0  s xxxx   .1.3.6.1.4.1.52330.6.2.1.0 i 0  .1.3.6.1.4.1.52330.6.2.5.0 i 1
```

When you look at the event list, you will see a new `unformatted enterprise event` for netsnmp_1_1 with the following contents:

![alt text](../session2/images/unformattedEvent.png "Figure unformattedEvent.png")

An `unformatted enterprise event` is an event OpenNMS displays when it receives a trap with an OID which it does not recognise. 

Conveniently, the event description contains a suggested event configuration which we could use to match against this trap.
We will now build an event configuration to match against our new trap.

### Event Configuration

The top level OpenNMS event configuration file is [etc/eventconf.xml](../../main/pristine-opennms-config-files/etc-pristine/eventconf.xml).
This can contain its own event configurations but can also reference a list of event configuration files in the [etc/events](../../main/pristine-opennms-config-files/etc-pristine/events) folder.

You will see that the [etc/events](../../main/pristine-opennms-config-files/etc-pristine/events) folder contains event definition files for the SNMP traps from many vendors with the vendor's prefixing the name of the event definition file.

It also contains a number of definitions for events generated internally by OpenNMS.
These event files are usually named with `opennms` as the file name prefix.

The order in which the event definition files are referenced in [etc/eventconf.xml](../../main/pristine-opennms-config-files/etc-pristine/eventconf.xml) is important.
The first matching event definition found when interpreting an event will always be used. 

For this reason, the last file in the list is always [etc/events/opennms.catch-all.events.xml](../../main/pristine-opennms-config-files/etc-pristine/events/opennms.catch-all.events.xml) because this defines the event definition we have already seen for a completely unknown trap.

Normally, we wouldn't want to alter a configuration directly inside a container because we will loose this when the container re-starts.
However as an experiment, we are going to modify the eventconf.xml inside our container to include a definition for this new trap.

The netsnmp containers already have the [vi editor](https://devhints.io/vim) installed but we will also install the [nano editor](https://www.nano-editor.org/dist/latest/nano.html) because it is easier to use.

```
# log into the opennms horizon container as root
docker compose exec -u root horizon bash

# install nano
root@horizon:/usr/share/opennms# apt-get install -y nano

# exit as root user 
```

now we can use nano to edit our configuration

```
# log into the opennms horizon container
docker compose exec horizon bash

# edit the eventconf.xml file
nano etc/eventconf.xml
```
you can now paste the following event definitions into the file between the end of the </global> definition and the event file list

```
   </global>

   <event>
      <mask>
         <maskelement>
            <mename>id</mename>
            <mevalue>.1.3.6.1.4.1.52330.6.2</mevalue>
         </maskelement>
         <maskelement>
            <mename>generic</mename>
            <mevalue>6</mevalue>
         </maskelement>
         <maskelement>
            <mename>specific</mename>
            <mevalue>1</mevalue>
         </maskelement>
         <varbind>
             <vbnumber>3</vbnumber>
             <vbvalue>1</vbvalue>
         </varbind>
      </mask>
      <uei>uei.opennms.org/traps/example_trap_definition_raise</uei>
      <event-label>example trap definition raise 1</event-label>
      <descr>example trap definition 1 varbind1=%parm[#1]% varbind2= %parm[#2]% varbind3= %parm[#3]%
      </descr>
      <logmsg dest="logndisplay">example trap definition 1 varbind1=%parm[#1]% varbind2= %parm[#2]% varfbind3= %parm[#3]%
      </logmsg>
      <severity>Warning</severity>
      <alarm-data reduction-key="%uei%:%dpname%:%nodeid%" alarm-type="1" auto-clean="false"/>
   </event>
   <event>
      <mask>
         <maskelement>
            <mename>id</mename>
            <mevalue>.1.3.6.1.4.1.52330.6.2</mevalue>
         </maskelement>
         <maskelement>
            <mename>generic</mename>
            <mevalue>6</mevalue>
         </maskelement>
         <maskelement>
            <mename>specific</mename>
            <mevalue>1</mevalue>
         </maskelement>
         <varbind>
             <vbnumber>3</vbnumber>
             <vbvalue>0</vbvalue>
         </varbind>
      </mask>
      <uei>uei.opennms.org/traps/example_trap_definition_clear</uei>
      <event-label>example trap definition clear 1</event-label>
      <descr>example trap definition 1 varbind1=%parm[#1]% varbind2= %parm[#2]% varfbind3= %parm[#3]%
      </descr>
      <logmsg dest="logndisplay">example trap definition 1 varbind1=%parm[#1]% varbind2= %parm[#2]% varfbind3= %parm[#3]%
      </logmsg>
      <severity>Normal</severity>
      <alarm-data reduction-key="%uei%:%dpname%:%nodeid%"
                  alarm-type="2"
                  clear-key="uei.opennms.org/traps/example_trap_definition_raise:%dpname%:%nodeid%"
                  auto-clean="false"/>
   </event>

   <event-file>events/opennms.snmp.trap.translator.events.xml</event-file>

```
Exit the editor and save eventconf.xml

now send an event into OpenNMS to reload the event configuration without restarting OpenNMS

```
bin/send-event.pl uei.opennms.org/internal/reloadDaemonConfig -p 'daemonName Eventd'
```


```
The total number of arguments received with the trap: 3.

They were:

.1.3.6.1.4.1.52330.6.2.1.0="xxxx" .1.3.6.1.4.1.52330.6.2.1.0="0" .1.3.6.1.4.1.52330.6.2.5.0="1"

Here is a "mask" element definition that matches this event, for use in event configuration files:

    <mask>
      <maskelement>
        <mename>id</mename>
        <mevalue>.1.3.6.1.4.1.52330.6.2</mevalue>
      </maskelement>
      <maskelement>
        <mename>generic</mename>
        <mevalue>6</mevalue>
      </maskelement>
      <maskelement>
        <mename>specific</mename>
        <mevalue>1</mevalue>
      </maskelement>
    </mask> 
```

