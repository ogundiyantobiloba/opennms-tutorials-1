# Session 7 Wrap Up

[Main Menu](../README.md) | [Session 7](../session7/README.md)

## Introduction

This final session will pull together what we have learnt and introduce a few new topics.

In this example we will introduce grafana and an nginx proxy

![alt text](../session7/images/examplenetwork3.png "Figure examplenetwork3.png")

## Nginx Proxy

OpenNMS can sit behind a proxy. 
In the attached configuration we have placed OpenNMS behind an NGINX proxy using self signed certificates

When you start the example you will see a home page at https://localhost/index.html

OpenNMS will be at https://localhost/opennms

Grafana will be at https://localhost/grafana

You will also be able to see NGINX statistics at http://localhost/nginx_status

## Nginx Data Collection

See 

collectd-configuration.xml
http-datacollection-config.xml
snmp-graph.properties.d/nginx-graph.properties

```
http://localhost/nginx_status
Active connections: 2 
server accepts handled requests
 54 54 261 
Reading: 0 Writing: 1 Waiting: 1 
```
