# DigitalOcean Site-to-Site VPN

Site-to-site VPNs help privately connect your on-premises environments with your cloud environments on DigitalOcean. They also facilitate secure inter-VPC communication, known as VPC peering. This guide demonstrates how to establish a site-to-site IPsec VPN with StrongSwan using a Droplet as a VPN gateway.

## Architecture

Imagine running workloads in two DigitalOcean data centers in different regions that need to communicate privately. These workloads run on backend Droplets, which we will connect over a VPN tunnel running between a pair of gateway Droplets, as illustrated in the diagram below.

![Architecture Diagram](https://lucid.app/publicSegments/view/24b18af7-7e06-4d37-b7f8-d117aae9f0e3/image.png)

A few notes about this architecture:
- It doesn't have to be StrongSwan on both sides of the VPN tunnel; any IPsec software/hardware gateway will do.
- It doesn't have to be DigitalOcean on both sides of the VPN tunnel; any other cloud provider or on-premises data center will work.
- It doesn't have to be just two sites; you can have multiple sites and establish a partial or full connection mesh.

This particular implementation uses IKEv2 and pre-shared keys. It also employs an always-on VPN tunnel, but feel free to use an on-demand tunnel with `auto=route` if it suits your needs better.

## Configuration

**strongswan-ams3** Droplet configuration (Ubuntu 22.04)
```
apt install strongswan -y
apt install net-tools -y

nano /etc/ipsec.conf

conn %default
	ikelifetime=60m
	keylife=20m
	rekeymargin=3m
	keyingtries=1
	authby=secret
	keyexchange=ikev2
	mobike=no

conn net-net
	leftid=@ams3
	leftfirewall=yes
	left=167.99.46.201
	leftsubnet=10.110.0.0/20

	right=64.225.7.37
	rightsubnet=10.108.0.0/20
	rightid=@nyc3
	auto=start

	type=tunnel
	dpdaction=restart

nano /etc/ipsec.secrets
@ams3 @nyc3 : PSK 'cm9ja2VyYm66ZXIK'

sudo ufw disable
sysctl -w net.ipv4.ip_forward=1

ipsec rereadsecrets
ipsec reload
ipsec status
```
**strongswan-nyc3** Droplet configuration (Ubuntu 22.04)
```
apt install strongswan -y
apt install net-tools -y

nano /etc/ipsec.conf

conn %default
	ikelifetime=60m
	keylife=20m
	rekeymargin=3m
	keyingtries=1
	authby=secret
	keyexchange=ikev2
	mobike=no

conn net-net
	leftid=@nyc3
	leftfirewall=yes
	left=64.225.7.37
	leftsubnet=10.108.0.0/20

	right=167.99.46.201
	rightsubnet=10.110.0.0/20
	rightid=@ams3
	auto=start

	type=tunnel
	dpdaction=restart

nano /etc/ipsec.secrets
@nyc3 @ams3 : PSK 'cm9ja2VyYm66ZXIK'

sudo ufw disable
sysctl -w net.ipv4.ip_forward=1

ipsec rereadsecrets
ipsec reload
ipsec status
```
**backend-ams3** Droplet configuration (Ubuntu 22.04)
```
apt install net-tools -y
ip route add 10.108.0.0/20 via 10.110.0.7 dev eth1
```
**backend-nyc3** Droplet configuration (Ubuntu 22.04)
```
apt install net-tools -y
ip route add 10.110.0.0/20 via 10.108.0.2 dev eth1
```
## Verification

To verify this setup, we will use Python to run an HTTP server on the backend-nyc3 Droplet. 

```
python3 -m http.server 8000
```

We will then access this HTTP server from backend-ams3 via a private network.

```
curl http://10.108.0.3:8000
```

This is the response you should expect to get.
```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>Directory listing for /</title>
</head>
<body>
    <h1>Directory listing for /</h1>
    <hr>
    <ul>
        <li><a href=".bashrc">.bashrc</a></li>
        <li><a href=".cache/">.cache/</a></li>
        <li><a href=".cloud-locale-test.skip">.cloud-locale-test.skip</a></li>
        <li><a href=".profile">.profile</a></li>
        <li><a href=".ssh/">.ssh/</a></li>
        <li><a href=".wget-hsts">.wget-hsts</a></li>
        <li><a href="snap/">snap/</a></li>
    </ul>
    <hr>
</body>
</html>
```

## Contact
Vasily Prokopov, Solutions Engineer at DigitalOcean – [vprokopov@digitalocean.com](mailto:vprokopov@digitalocean.com).

If you wish to learn more about DigitalOcean's services, you are welcome to reach out to the sales team at [sales@digitalocean.com](mailto:sales@digitalocean.com). A global team of talented engineers will be happy to provide assistance.
