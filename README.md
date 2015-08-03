# nodejs arp-discovery
Discover hosts in the local network using ARP.

Tested on Mac and Linux (ubuntu).

It uses the command "arp -a" to get hosts
for the LAN. Before running the command it
floods the local network with requests for all
IP addresses, hopefully getting the ARP table
filled up with goodies.

All mac addresses are normalized to uppercase
and leadings "0" if needed.

# Usage
## Parameters

    {
      max_connections:    64,
      macvendor_api:      'http://api.macvendors.com/',
      resolve_macvendor:  false,
      max_hosts:          1024,
      timeout:            2500,
      port:               1,
      flood_interval:     300,  
      restrict:           null,
    }

* max\_connections: poll this many hosts at once (default: 64)
* macvendor_api: api to resolve vendor names (default: 'http://api.macvendors.com/')
* resolve_macvendor: enable vendor names resolution, only with discover() (default: false)
* max\_hosts: the maximum number of IP addresses to check per subnet
* timeout: flood connections timeout ms (defaut:2500)
* port: flood hosts on this port (not important, since it just trigger ARP request)
* flood_interval: flood interval (sec) default 5 minutes (default:300)
* restrict: only scan subnets that include this IP. Set to "false" if you
  don't want restrictions, otherwise it will only scan the first interface

## Code

      var util          = require('util')
      ,   ARP_Discovery = require('arp-discovery')
      ;

      var arp = new ARP_Discovery({timeout:1000, flood_interval: 300, resolve_macvendor:true});

      arp.on('error', function(err){
        console.log(err);
      });

      // Discovery :
      arp.on('success', function(res) {
          if (!res) {
              console.log('!res');
              return;
          }
          console.log('success: '+util.inspect(res, { depth: null }));
          console.log('getMacs: '+util.inspect(arp.getMacs(), { depth: null }));
          console.log('getIps:  '+util.inspect(arp.getIps(), { depth: null }));
      });
      arp.discover();

      // Monitoring :
      arp.on('lost', function(lost){
        console.log('lost: '+util.inspect(lost, { depth: null }));
      });

      arp.on('found', function(found){
        console.log('found: '+util.inspect(found, { depth: null }));
      });

      arp.on('update', function(update){
        console.log('update: '+util.inspect(update, { depth: null }));
      });

      // Monitoring interval (ms)
      arp.monitor(40000);


## Sample Results

    getMacs:
    {
        '20:AA:4B:CB:63:47': '192.168.24.1',
        '00:30:DE:08:B1:75': '192.168.24.100',
        '30:05:5C:6B:13:EF': '192.168.24.169',
        'A4:5E:60:E7:63:B5': '192.168.24.183',
        '00:01:2E:4D:3F:86': '192.168.24.192',
        '00:18:DD:41:02:6A': '192.168.24.216',
        'B4:18:D1:DE:06:3B': '192.168.24.245' 
    }
    found / lost / update:
    { 
        ip: '192.168.24.245',
        host: 'airport-express-de-ana.lan',
        mac: 'B4:18:D1:DE:06:3B',
        interface: 'en0',
        seen: 1438530008346 
    }
    success:
    { 
        '20:AA:4B:CB:63:47':
           { 
                ip: '192.168.24.1',
                host: 'openwrt.lan',
                mac: '20:AA:4B:CB:61:41',
                interface: 'en0',
                vendor:'Cisco system',
                seen: 1438530008336 
            },
        '00:30:DE:08:B1:75':
           { 
                ip: '192.168.24.100',
                host: 'unknown',
                mac: '00:30:DE:08:B1:75',
                interface: 'en0',
                vendor: 'Apple',
                seen: 1438530008341 
            }
    }
