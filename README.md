# UDP multicast

#### UDP multicast MPEG-TS streams on Linux

> A most common task for all OTT and IPTV video services is, of course, to encode a group of TV channels. And the most common way you will be receiving channels is MPEG-TS streams via multicast UDP. In this article we have explained some popular tips and tricks for working with UDP multicast MPEG-TS streams on Ubuntu and CentOS. We assume you have a linux server or a workstation with a root (superuser) access and a bunch of multicast streams available in your network.

#### Checking a multicast UDP stream
Suppose you know incoming UDP MPEG-TS multicast host and port:

```sh
udp://233.0.41.102:20000
```
In this URL 233.0.41.102 is called IP address (or host) and 20000 is port. We will use these later as an example for configuration settings.

### Using tcpdump

Let’s make sure we are actually receiving a stream using [tcpdump](http://www.tcpdump.org/manpages/tcpdump.1.html). You must be `root` to run `tcpdump`. Execute the following command:

```sh
> tcpdump -c 10 dst host 233.0.41.102 and port 20000 and multicast
```
This command tells to capture 10 multicast UDP packets sent to host 233.0.41.102 and port 20000. If the stream is running, you shall get something like:


```sh
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
15:20:58.818832 IP 43.232.net.50000 > 233.0.41.102.20000: UDP, length 1316
15:20:58.824080 IP 43.232.net.50000 > 233.0.41.102.20000: UDP, length 1316
15:20:58.829467 IP 43.232.net.50000 > 233.0.41.102.20000: UDP, length 1316
15:20:58.835816 IP 43.232.net.50000 > 233.0.41.102.20000: UDP, length 1316
15:20:58.841053 IP 43.232.net.50000 > 233.0.41.102.20000: UDP, length 1316
15:20:58.846617 IP 43.232.net.50000 > 233.0.41.102.20000: UDP, length 1316
15:20:58.852052 IP 43.232.net.50000 > 233.0.41.102.20000: UDP, length 1316
15:20:58.857893 IP 43.232.net.50000 > 233.0.41.102.20000: UDP, length 1316
15:20:58.863787 IP 43.232.net.50000 > 233.0.41.102.20000: UDP, length 1316
15:20:58.869017 IP 43.232.net.50000 > 233.0.41.102.20000: UDP, length 1316
10 packets captured
70 packets received by filter
4 packets dropped by kernel
```
If you don’t see output and `tcpdump` seems to freeze, you are not receiving multicast packets. Press `Ctrl-C` to abort `tcpdump` and refer to the next section on firewall settings.

#### Using netstat
An alternative way to check multicast stream is by using `netstat` tool. You also need to be `root` to use it.

```sh
> netstat -a -u -n | grep 233.0.41.102:20000
```
If multicast stream is running, you will see:
```sh
udp        0      0 233.0.41.102:20000      0.0.0.0:*
```
(this line may appear twice or more times)
If previous command emits nothing, you are not receiving multicast packets. Refer to the next section on firewall settings.

#### Firewall and multicast issues with multicast
Very often multicast streams are blocked by firewall. There are various firewalls on linux distributives, we will cover the most popular: `ufw` (Uncomplicated FireWall) for Ubuntu and `iptables` for various distributives.

Be advised that by misconfiguring the firewall **you may be unable to access your server via SSH!** Disabling firewall at all is highly insecure and **not recommended.**

#### ufw on Ubuntu
`ufw` is a default firewall for Ubuntu linux. To operate ufw you must be root.

Check current status of `ufw`:
```sh
> ufw status
```

If `ufw` is active, add rule to enable receiving multicast on specified host and port:

```sh
> ufw allow to 233.0.41.102 port 20000 proto udp
```

You may completely disable `ufw` by typing:

```sh
> ufw disable
```
This will allow all incoming connections which is insecure and really **not recommended.** Use it only for testing purposes for a limited period of time.

More info about ufw: `UFW` on [Ubuntu](https://help.ubuntu.com/community/UFW)

#### iptables
`iptables` is a more complicated firewall and it could be found on Ubuntu, CentOS and many more linux systems. To operate `iptables` you must be root.

To check `iptables` status run:

```sh
> iptables -L
```
It will show status and list of the firewall rules. If you have a fresh installation of OS, you should see empty rule chains:

```sh
Chain INPUT (policy ACCEPT) target     prot opt source               destination

Chain FORWARD (policy ACCEPT) target     prot opt source               destination

Chain OUTPUT (policy ACCEPT) target     prot opt source               destination
```

To allow an incoming multicast stream you need to add a rule by running the following command:

```sh
> iptables -A INPUT -p udp -d 233.0.41.102 --dport 20000 -j ACCEPT
```
Run `iptables` -L again and you will see:

```sh
ACCEPT     udp  --  anywhere             233.0.41.102         udp dpt:20000
```

`iptables` does not save its status after reboot, so you will need to save newly added rules by typing:

```sh
> iptables-save
```
More info on `iptables:`

- [Ubuntu iptables how-to](https://help.ubuntu.com/community/IptablesHowTo)
- [CentOS iptables how-to](https://wiki.centos.org/HowTos/Network/IPTables)

#### Getting a list of all multicast streams
Sometimes you don’t have list of all multicast streams available. You may get a list of all incoming multicasts quickly by running following simple command:

```sh
> sudo tcpdump -n -c 100000 multicast | perl -n -e 'chomp; m/> (\d+.\d+.\d+.\d+).(\d+)/; print "udp://$1:$2\n"' | sort | uniq
```
It may take some time to capture 100000 packets and produce output — press `Ctrl-C` if you are tired and reduce amount of packets to capture.

This will produce a sorted list of incoming multicasts like:

```sh
udp://233.0.37.102:20000
udp://233.0.41.102:20000
udp://233.0.42.100:20000
udp://233.0.42.106:20000
udp://233.0.58.114:20000
udp://233.0.58.170:20000
```
To make sure it is an actual MPEG-TS stream and read it parameters refer to the next section.

#### Checking multicast MPEG-TS with ffprobe
`ffprobe` is a versatile utility to check various type of media files and streams. It comes in `ffmpeg` package.

Install `ffmpeg` (on Ubuntu):

```sh
> sudo apt-get install ffmpeg
```
Install `ffmpeg` (on CentOS):
```sh
> sudo yum install ffmpeg
```
Use `ffprobe` to check multicast stream:

```sh
> ffprobe udp://233.0.41.102:20000
```
For a valid MPEG-TS stream you will get something like this:

```sh
Input #0, mpegts, from 'udp://233.0.41.102:20000':
 Duration: N/A, start: 89651.690400, bitrate: 192 kb/s
 Program 10106
   Metadata:
     service_name    : AChannel
     service_provider: ChannelProvider
   Stream #0:0[0x19a][6]: Audio: mp2 (\[4\]\[0\]\[0\]\[0\] / 0x0004), 48000 Hz, stereo, s16p, 192 kb/s (clean effects)
   Stream #0:1[0x145]: Video: mpeg2video (Main) (\[2\]\[0\]\[0\]\[0\] / 0x0002), yuv420p(tv), 720x576 [SAR 64:45 DAR 16:9], max. 15000 kb/s, 25 fps, 25 tbr, 90k tbn, 50 tbc
```
Note lines starting with Stream, they contain information about video and audio data: codecs, bitrate, fps etc.
