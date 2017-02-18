## Using a forge module - An NTP example

This page will give you a first hand example of configuring a NTP service ( as a server or client ) through Puppet. There's a pretty decent quick start guide [here](https://docs.puppet.com/pe/latest/quick_start_ntp.html) from Puppet itself. You may want to look at that first. If it serves your purpose, you need not read any further here. 

_Note_ :
_This doc is for a different audience than the quick start guide. It is for someone who has Puppet installed and has a basic idea on how Puppet works, has seen some basic puppet code. He/she knows what a class, module and resource in puppet is, but doesn't have much experience using it._

### NTP - the manual way

So, let's first setup NTP manually and see what exactly are we automating. If you know manual setup of NTP, you can skip to next section.

#### A little about NTP in Linux

So, NTP ( Network Time Protocol ) is way to ensure your machines are in sync w.r.t time. That's all ! 

The NTP server listens on port 123 and uses UDP. The NTP clients send requests to NTP server to sync its time with it. The config file used by both NTP server and client is _/etc/ntp.conf_ ( for RedHat and Ubuntu systems ). The NTP service ( called _ntpd_ ) acts as a server and/or a client depending on its configuration.

There are two packages that are most commonly talked about in NTP :

1. _ntp_ package, which sets up the _ntpd_ service. 

    The _ntpd_ service runs in the background as a deamon and queries the configured NTP servers periodically to keep time in sync. It will also act as a server if it is configured to accept requests.
    
2. _ntpdate_ package, which is to manually sync time with a NTP server.

    The _ntpdate_ package gives a command of same name to query NTP servers.
    
#### Setting up NTP in RedHat based system.

Let's setup a NTP server and a NTP client. For both we need the same package _ntp_ and package _ntpdate_. Let's install it.

```
[On Server & client]
# rpm -qa | grep ntp
#

# yum install ntp ntpdate
#

# rpm -qa | grep ntp
ntpdate-4.2.6p5-25.el7.centos.1.x86_64
ntp-4.2.6p5-25.el7.centos.1.x86_64
#

```

Get the ipaddress for both Server and Client
```
[On server]
[root@codemaster ~]# facter ipaddress
192.168.138.131

[On client]
[root@centos7agent ~]# facter ipaddress
192.168.138.129
```

##### Configuring the NTP server
   
On the machine you want to make as the NTP server, edit the _/etc/ntp.conf_ as follows :

```
[root@codemaster ~]# vim /etc/ntp.conf

# Search for line "restrict" and add something like the following, to allow traffic from local IPs.
    
    restrict 127.0.0.1
    restrict ::1
    restrict 192.168.138.0 mask 255.255.255.0 nomodify notrap   # this is the line that makes it an NTP server.

# We are basically asking the "ntpd" service to start listening and responding to requests from a certain set of IPs.
# Explanation about IP values : 
# My client nodes are in the IP range 192.168.138.0-255. Hence, the above IP and mask. 
# You could give a broader range if you need to. like 192.168.0.0 & mask 255.255.0.0

```

Your local NTP server would typically connect to some other external NTP server like below, to keep its time in sync. SO, the ntp.conf would also have some entry like this :

```
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
```
 
That's it. Start the _ntpd_ service to start the NTP server. 






