# 搭建VPN
## 新建 EC2 Instance
注意选择security group

[Connecting to Your Linux Instance Using SSH](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)

```
ssh -i /path/my-key-pair.pem ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com
```

## 安装 OpenVPN
[OpenVPN Downloads](https://openvpn.net/index.php/open-source/downloads.html)

### Linux (yum)

```
sudo yum install -y openvpn
```


### Linux (without RPM)

```
tar xfz openvpn-[version].tar.gz
```

Then `cd` to the top-level directory:

```
./configure
make
sudo make install
```

### Linux (using RPM package)

To check if `rpm-build` installed:

```
rpm rpm-build -q
```

Install `rpm-build`:

```
sudo yum install rpm-build
```

Dependencies:

```
sudo yum install openssl-devel
sudo yum install lzo-devel
sudo yum install pam-devel
```

Build your own RPM file, if you don't have it:

```
rpmbuild -tb openvpn-[version].tar.gz
```

Once you have the .rpm file, you can install it with the usual

```
rpm -ivh openvpn-[details].rpm
```

or upgrade an existing installation with

```
rpm -Uvh openvpn-[details].rpm
```

## Static key configuration

Generate a static key:

```
openvpn --genkey --secret static.key
```

Copy the static key to both client and server.

Server configuration file:

```
dev tun
ifconfig 10.8.0.1 10.8.0.2
secret static.key
```

On server side, use a command to NAT the VPN client to the Internet:

```
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
```

Start `openvpn` server:

```
sudo openvpn --config server.conf --daemon
```

Client configuration file:

```
remote myremote.mydomain
dev tun
ifconfig 10.8.0.2 10.8.0.1
secret static.key
redirect-gateway def1
```

Use [Tunnelblick](https://tunnelblick.net/) to open `openvpn` client.

([OpenVPN HOWTO - Route all client traffic through the VPN](https://openvpn.net/index.php/open-source/documentation/howto.html#redirect))


## Setting up your own Certificate Authority (CA) and generating certificates and keys for an OpenVPN server and multiple clients

### Easy-RSA 3 Setup

Refer to [](https://github.com/OpenVPN/easy-rsa/blob/master/README.quickstart.md)
