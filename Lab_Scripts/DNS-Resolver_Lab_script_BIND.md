# DNS Resolver Lab (using BIND & Unbound)

------

> (2023-10-10) 

------



## Lab Topology (Group X) 



![lab-topo-resolver-authoritative](./DNS-Resolver_Lab_script-pics/grpX_network_topology.png)



```
  DEVICE NAME        IPv4 ADDRESS              IPv6 ADDRESS
+--------------+-----------------------+-----------------------------+
| grpX-cli     | 100.100.X.2 (eth0)    | fd52:a627:X::2 (eth0)       |
+--------------+-----------------------+-----------------------------+
| grpX-resolv1 | 100.100.X.67 (eth0)   | fd52:a627:X:64::67 (eth0)   |
+--------------+-----------------------+-----------------------------+
| grpX-resolv2 | 100.100.X.68 (eth0)   | fd52:a627:X:64::68 (eth0)   |
+--------------+-----------------------+-----------------------------+
| grpX-rtr     | 100.64.1.X (eth0)     | fd52:a627:X::1 (eth1)       |
|              | 100.100.X.65 (eth2)   | fd52:a627:X:64::1 (eth2)    |
|              | 100.100.X.193 (eth4)  | fd52:a627:X:192::1 (eth4)   |
|              | 100.100.X.129 (eth3)  | fd52:a627:X:128::1 (eth3)   |
|              | 100.100.X.1 (eth1)    | fd52:a627:0:1::X (eth0)     |
+--------------+-----------------------+-----------------------------+
```

During this practice we are only going to access the following equipment:

* **grpX-cli** : client (optional)
* **grpX-resolv1**: recursive server (resolver) with preinstalled BIND



# Setting up recursive server (BIND)

We use the container "*resolv1*" (recursive server) [**grpX-resolv1**].

This container already has the BIND9 packages downloaded and installed.

We switch to the root user:

```
$ sudo su -
```

We go to the /etc/bind directory:

```
# cd /etc/bind
```

At this point we must configure some BIND9 options.
To do this we edit the file /etc/bind/named.conf.options:

```
# nano named.conf.options
```

Now we add the options to indicate (when resolving) which are the IP addresses that will be able to send DNS queries and at the same time to which IP addresses our server will listen on port 53 (in this case both prefixes are identical). The file should be as follows:

```
options {
	directory "/var/cache/bind";

	// If there is a firewall between you and nameservers you want
	// to talk to, you may need to fix the firewall to allow multiple
	// ports to talk. See http://www.kb.cert.org/vuls/id/800113

	// If your ISP provided one or more IP addresses for stable 
	// nameservers, you probably want to use them as forwarders.  
	// Uncomment the following block, and insert the addresses replacing 
	// the all-0's placeholder.

	// forwarders {
	// 	0.0.0.0;
	// };

	//========================================================================
	// If BIND logs error messages about the root key being expired,
	// you will need to update your keys. See https://www.isc.org/bind-keys
	//========================================================================
	dnssec-validation auto;

	listen-on port 53 { any; };																		<--- Add this
	listen-on-v6 port 53 { any; };																<--- Add this
	
	allow-query { localhost; 100.100.0.0/16; fd52:a627::/32; };		<--- Add this

	recursion yes;																								<--- Add this
};
```

Once we finish editing the configuration file, we execute a command that allows us to quickly check if the configuration is semantically correct (if the command does not return anything, it means that it did not find errors in the configuration files):

```
# named-checkconf
```

Finally we restart the server so that it takes the configuration changes:

```
# systemctl restart bind9
```

And we check the status of the bind9 process:

```
# systemctl status bind9
```

We should obtain an output similar to the following:

```
● named.service - BIND Domain Name Server
   Loaded: loaded (/lib/systemd/system/named.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/service.d
       └─lxc.conf
   Active: **active (running)** since Thu 2021-05-13 01:38:27 UTC; 4s ago
    Docs: man:named(8)
  Main PID: 849 (named)
   Tasks: 50 (limit: 152822)
   Memory: 103.2M
   CGroup: /system.slice/named.service
       └─849 /usr/sbin/named -f -u bind

May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: **command channel listening on ::1#953**
May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: managed-keys-zone: loaded serial 6
May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: zone 0.in-addr.arpa/IN: loaded serial 1
May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: zone 127.in-addr.arpa/IN: loaded serial 1
May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: zone localhost/IN: loaded serial 2
May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: zone 255.in-addr.arpa/IN: loaded serial 1
May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: **all zones loaded**
May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: **running**
May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: managed-keys-zone: Key 20326 for zone . is now trusted (acceptance timer>
May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: resolver priming query complete
```

 
