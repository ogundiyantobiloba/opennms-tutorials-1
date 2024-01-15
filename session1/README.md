# Session 1 Introduction to OpenNMS

## Contents

1. What is OpenNMS
2. Getting Started Tutorial
* Installation
* Running docker compose example 1
* Importing a network - discovery
* SNMP community strings
* Importing a network - requisitions

## What is OpenNMS

### Overview

OpenNMS is the world first enterprise and service provider grade service assurance platform created using the open source model.
OpenNMS was first published in the [sourceforge opennms project](https://sourceforge.net/projects/opennms/) over 20 years ago. 
More recently, it has been maintained in the official [github opennms project](https://github.com/OpenNMS/opennms)
Since that time the number of users and contributors has grown into a world wide community of people who use OpenNMS for a variety of purposes.

* service providers - use multiple instances of OpenNMS as a service assurance components within in their operations support stack
* large enterprises - rely on on OpenNMS as their primary netowrk monitoring platform for  very large scale enterprise networks
* small enterprises - significnalty reduce the work load for their IT team by monitoring their networks and services using OpenNMS
* OEM vendors - package OpenNMS as an IP mediation layer within their own Network or Element management solution set. 

An overview of the major capabilities of OpenNMS is provided in the following slide presentation [OpenNMS_Overview-v1.0.pdf](../session1/OpenNMS_Overview-v1.0.pdf)

### Documentation
Comprehensive documentation on using the OpenNMS project is provided through the [OpenNMS Documentation Site](https://docs.opennms.com/start-page/1.0.0/index.html).

You will see that documentation is available for both the Meridian and Horizon distributions of OpenNMS.

Horizon is the cutting edge, fast moving (but unsupported) distribution of OpenNMS.
New major releases of Horizon come out every 3-6 months.
Horizon will always contain the latest features as it is the OpenNMS development stream.
However, there is no guarantee of backwards compatibility of API's or configurations between Horizon releases.

Meridian is the hardened long term supported version of OpenNMS. 
New major releases of Meridian come out once a year and each major release is supported for 3 years through point releases providing bug fixes or security patches.
API's and configurations devloped for a given Meridian release are guaranteed not to require updating over the 3 year life of the platform.
This makes Meridian a more suitable choice for a production environment.

This tutorial and it's examples have currently been designed to work with Horizon 32 but the principles covered will also apply to Meridian 2023.

As you work through these tutorials you should also familiarise yourself with the relevent documentation of each feature covered as the documentation will provide more details of options which aren't included in the examples and also any minor differences between Meridian and Horizon.

### Other Sources of Information

Over the years, many contributors have added insights or examples and these are often available by searching the [OpenNMS Discourse Server](https://opennms.discourse.group/).

In addition, the [OpenNMS Mattermost chat](https://chat.opennms.com/) channels are a great place to ask questions of the community.

## Getting Started Tutorial





