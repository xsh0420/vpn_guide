# Setup VPN
## Launch EC2 Instance
Set security group

[Connecting to Your Linux Instance Using SSH](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)

```
ssh -i /path/my-key-pair.pem ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com
```

## Install OpenVPN
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

## a. Static key configuration

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


## b. PKI Infrastructure

### Easy-RSA 3 Setup

Refer to [Easy-RSA 3 Quickstart README](https://github.com/OpenVPN/easy-rsa/blob/master/README.quickstart.md)

1. Choose a system to act as your CA and create a new PKI and CA:

        ./easyrsa init-pki
        ./easyrsa build-ca

2. On the system that is requesting a certificate, init its own PKI and generate a keypair/request. Note that the init-pki is used only when this is done on a separate system (or at least a separate PKI dir.) This is the recommended procedure. If you are not using this recommended procedure, skip the next import-req step as well.

        ./easyrsa init-pki
        ./easyrsa gen-req EntityName

3. Transport the request (.req file) to the CA system and import it. The name given here is arbitrary and only used to name the request file.

        ./easyrsa import-req /tmp/path/to/import.req EntityName

4. Sign the request as the correct type. This example uses a client type:

        ./easyrsa sign-req client EntityName

5. Transport the newly signed certificate to the requesting entity. This entity may also need the CA cert (ca.crt) unless it had a prior copy.

6. The entity now has its own keypair, and signed cert, and the CA.

### Generate a shared-secret key for tls-auth

    openvpn --genkey --secret ta.key

Copy `ta.key` to both sides.

### Generate Diffie hellman parameters (server side):

    openssl dhparam -out dh2048.pem 2048

### Use unified form in the profile

    <ca>
    -----BEGIN CERTIFICATE-----
    MIIBszCCARygAwIBAgIE...
    . . .
    /NygscQs1bxBSZ0X3KRk...
    Lq9iNBNgWg==
    -----END CERTIFICATE-----
    </ca>

    <cert>
    -----BEGIN CERTIFICATE-----
    . . .
    </cert>

    <key>
    -----BEGIN RSA PRIVATE KEY-----
    . . .
    </key>

    key-direction 1
    <tls-auth>
    -----BEGIN OpenVPN Static key V1-----
    . . .
    </tls-auth>

`key-direction 0` for server.


### Start server using `--askpass` option

    sudo openvpn --config server.ovpn --daemon --askpass key.pass
