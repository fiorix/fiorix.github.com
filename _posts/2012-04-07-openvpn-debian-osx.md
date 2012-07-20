---
layout: post
title: OpenVPN between Debian and OSX
location: Montréal, Canada
comments: true
tags:
 - openvpn
 - udp tunnel
 - debian
 - osx
 - security
 - roaming
---

I've been travelling a lot these days. What pisses me off the most when I'm
roaming, is the number of different limitations on Internet access.

Although you can easily buy pre-paid SIM cards almost everywhere, some of them
ship with non-sense limitations, like Middle East telecoms blocking Skype for
profit, operators in the UK miscounting your usage, not to mention customer's
offices where you're required to browse through broken and very limited HTTP
proxies with absolutely no privacy, nor access to other protocols such as ssh.

Over all these years, I've seen many different creative ways of limiting
people's access to Internet, and, many holes that are left behind, that we can
explore to get full access to the network, the way we like it.
I'd say that, in general, most networks let go through:

- DHCP/BOOTP traffic on UDP ports 67 and 68
- SNMP and SNMP Traps on UDP and TCP ports 161 and 162, respectively
- DNS traffic, on UDP and TCP port 53
- HTTPS traffic on TCP port 443, because it can't (or shouldn't) be proxied

Thus, the easiest way to bypass most of the limitations, and some times even
get free (out of charge) traffic, is by setting up a tunnel from your computer
or cell phone to a remote server, using one of the protocols (TCP, or UDP) and
port numbers mentioned above.

There are way too many options out there. From super simple set ups using
[PPTP](http://poptop.sourceforge.net/), to super hax0r tools like
[iodine](http://code.kryo.se/iodine/) - pretty damn slow, by the way.

Usually, I keep a couple of PPTP servers in different countries, so I can tunnel
traffic from my computer, phones or tablets wherever I am. However, PPTP is not
always allowed - in fact, it's almost always blocked on restrictive networks.

The solution that fits me better these days is [OpenVPN](http://openvpn.net/).
It's been out there for a while, and is supported on most popular operating
systems. The setup is very simple, it supports both UDP and TCP as transport,
and works on whatever port number you choose. That means that even on the most
restrictive networks, you may find holes to set up your tunnel on, and OpenVPN
is usually the best option.


### Setting up the server

This is how to set up the OpenVPN on a debian server. First, install it:

    apt-get install openvpn

Now, let's prepare the config. OpenVPN supports routed (tun) or bridged
(tap) connections. Basically, routed is when you have a network device on each
endpoint with routing between them, as opposed to the bridged mode, where
OpenVPN will make all of your endpoints see others as if they were on the same
network segment - with all the broadcast, etc. We want routed mode for tunnels.

Next step is to create server and client certificates for authentication
purposes. For that, we'll start by creating a CA, then the server certificate,
and finally as many client certificates you want, all signed by the same CA.

The debian package ships with scripts to help generating and maintaining the CA
and its certificates. We'll make a copy of it into the configuration directory,
to keep our own settings separate from the package files.

    cp -r /usr/share/doc/openvpn/examples/easy-rsa/2.0 /etc/openvpn/easy-rsa
    cd /etc/openvpn/easy-rsa
    source ./vars
    ./clean-all
    ./build-ca

That will set up the CA for signing both server and client certificates.
Next step is to create the server certificate, and sign it. At the end of the
process you'll be asked to sign the certificate. Make sure you answer *yes*.

    ./build-key-server server

Now, let's create the diffie-hellman parameters, which are used for the key
exchange between server and client.

    ./build-dh

That is all you need for the server side. Finally, create the first client
certificate. Again, you'll be asked to sign the certificate at the end of the
process. Make sure you answer *yes*.

    ./build-key-pkcs12 client1

You can repeat this process to create as many client certificates you want.
Next time you need a client certificate, make sure you run
<code>source ./vars</code> before calling
<code>./build-key-pkcs12 clientN</code>.

These scripts generate files in <code>/etc/openvpn/easy-rsa/keys</code>.

Now that we have the keys, let's create the server configuration file. Use your
preferred text editor to create <code>/etc/openvpn/server.conf</code> with
these settings:

    port 443
    proto tcp
    dev tun
    ca /etc/openvpn/easy-rsa/keys/ca.crt
    cert /etc/openvpn/easy-rsa/keys/server.crt
    key /etc/openvpn/easy-rsa/keys/server.key
    dh /etc/openvpn/easy-rsa/keys/dh1024.pem
    server 10.8.0.0 255.255.255.0
    ifconfig-pool-persist ipp.txt
    keepalive 10 120
    comp-lzo
    max-clients 5
    persist-key
    persist-tun
    status /var/log/openvpn-status.log
    verb 3
    push "redirect-gateway"

Please refer back to the list of protocols and port numbers I mentioned earlier,
and adjust the server settings to your needs. I usually run the server on UDP
port 53 in order to bypass most network restrictions.

Also, attention to the last line, <code>push "redirect-gateway"</code>, which
tells OpenVPN clients to install a default route (the default gateway) pointing
to this server. Feel free to comment or remove that line if that's not what you
want.

Now, start the server:

    /etc/init.d/openvpn start

If clients are using this server as their default gateway, they'll need NAT
to browse through this server. Don't forget to set up IP forwarding and
masquerading:

    echo 1 > /proc/sys/net/ipv4/ip_forward
    iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -j MASQUERADE


### Setting up the client

I've been using [Tunnelblick](http://code.google.com/p/tunnelblick/) on Mac OSX,
and it works just fine.

It does not have any fancy interface for configuring the VPN. Instead, it'll
ask you to create the configuration file, which is similar to the server one,
and import it along with the CA crt and client certificate used to authenticate
on the server.

The easiest way I found is to create a directory with all the files needed by
Tunnelblick, in a way similar to those Cisco VPN clients on Windows.

Suppose your server's host name is *my.server.com*. Then, create a directory
named <code>my.server.com.tblk</code>, and copy the following files from the
server into this directory on your mac:

    /etc/openvpn/easy-rsa/keys/ca.crt
    /etc/openvpn/easy-rsa/keys/client1.crt
    /etc/openvpn/easy-rsa/keys/client1.key

Create a file named <code>config.ovpn</code> in the same directory, with
the following settings:

    client
    dev tun
    proto tcp
    remote my.server.com 443
    resolv-retry infinite
    nobind
    persist-key
    persist-tun
    ca ca.crt
    cert client1.crt
    key client1.key
    comp-lzo
    verb 3

Adjust *proto* and *remote* to match your server settings. The directory should
look like this:

    $ ls -l my.server.com.tblk/
    total 32
    -rw-r--r--  1 bozo  staff  1208  4 Apr 06:44 ca.crt
    -rw-r--r--  1 bozo  staff  3718  4 Apr 06:44 client1.crt
    -rw-------  1 bozo  staff   887  4 Apr 06:44 client1.key
    -rw-r--r--  1 bozo  staff   162  4 Apr 06:44 config.ovpn

Once Tunnelblick is installed, open Finder and double click this directory. It
will automatically import the settings into Tunnelblick. You'll find
Tunnelblick's icon on the right side of the top menu bar. Just click
*connect my.server.com* and break free.
