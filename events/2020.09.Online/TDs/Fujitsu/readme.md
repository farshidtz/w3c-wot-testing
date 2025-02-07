# Fujitsu - TPAC 2020 Online Plugfest

Fujitsu provides two services:

- Sensor units (temperature, humidity, air pressure, human detection, etc.)
- A local proxy with local discovery

<img src="plugfest.png" width=50%>


## Sensor units

Several sensors and Wi-Fi module(ESP32) are on a single board that has a WoT interface.
In this plugfest, 2 different types of TDs for this sensor unit are provided. One is the TD for the physical device that can be accessed directly, and another is the TD of the virutal devicefor generated by the local proxy described below.

### discovery of this device

search `_wot._tcp` service on plugfest VPN.

```
% avahi-browse -r _wot._tcp
+ vpn_vpn IPv4 fujitsu-sensor                                _wot._tcp            local
= vpn_vpn IPv4 fujitsu-sensor                                _wot._tcp            local
   hostname = [ESP32-3D71BF428EFC.local]
   address = [192.168.30.139]
   port = [80]
   txt = ["type=Thing" "td=/Things/td"]
```

### retrieving TD from this device

```
% curl http://192.168.30.139/Things/td
```


## Local proxy

The proxy is allinged to the intermediary specified in the WoT achitecture document. It can aggregate multiple WoT devices and maintain the device information inside.
The proxy can keep TDs of things that cannot be accessed from others due to suspended or sleeping, and can behave them instead of the physical devices.

<img src="proxy.png" width=40%>

This proxy has supported the local discovery mechanism using mDNS. You can find the detail interface and sequence diagram in https://github.com/w3c/wot-discovery/blob/master/prior-work/fujitsu/README.md.

### discovery of local proxy

To search this proxy, browse `_wot._tcp` service on plugfest vpn.

```
% avahi-browse -r _wot._tcp
+ vpn_vpn IPv4 wot-directory                                 _wot._tcp            local
= vpn_vpn IPv4 wot-directory                                 _wot._tcp            local
   hostname = [ip-192-168-10-us-west-2-compute-internal.local]
   address = [192.168.30.10]
   port = [80]
   txt = ["register=/Things/register" "retrieve=/Things"]
```

This information shows you how to register and retrieve TDs to/from the proxy based on the "txt" information. the examples of registry and retrieving are below.

### registering a new divice

To register a new devices, post to register the TD of the device to the URL that crated by 'hostname' and 'register' field of 'txt'.

```
curl -X POST -H 'content-type: application/json' -d '<TD>' http://ip-192-168-30-10-us-west-2-compute-internal.local/Things/register
```
or
```
curl -X POST -H 'content-type: application/json' -d '<TD>' http://192.168.30.10/Things/register
```
The virtual device is generated on the proxy.

### retrieving registered TDs (Both of physical and virtual)

To get device list of registered IDs, the proxy returns the list of IDs assinged to TDs. An example is below. 

```
curl http://192.168.30.10/Things
```

To get a TD, choose one of IDs retuened from the proxy and post to the URL of the ID you choose.
If you get the virtual device, describe below.

```
curl http://192.168.30.10/Things/ID?type=shadow
```
or
```
curl http://192.168.30.10/Things/ID
```

If you get the physical device, post with 'type=original',

```
curl http://192.168.30.10/Things/ID?type=orignal
```
