# DigitalOcean Site-to-Site VPN

Site-to-site VPNs help privately connect your on-premises environments with your cloud environments on DigitalOcean. They also facilitate secure inter-VPC communication, known as VPC peering. This guide demonstrates how to establish a site-to-site IPsec VPN with StrongSwan using a Droplet as a VPN gateway.

## Architecture

Imagine running workloads in two DigitalOcean data centers in different regions that need to communicate privately. These workloads run on backend Droplets, which we will connect over a VPN tunnel running between a pair of gateway Droplets, as illustrated in the diagram below.

![Architecture Diagram](https://lucid.app/publicSegments/view/24b18af7-7e06-4d37-b7f8-d117aae9f0e3/image.png)

**A few notes about this architecture:**

- It doesn't have to be StrongSwan on both sides of a VPN tunnel; any IPsec software/hardware gateway will do.
- It doesn't have to be DigitalOcean on both sides of a VPN tunnel; any other cloud provider or on-premises datacenter will work.
- It doesn't have to be just two sites; you can have multiple sites and establish a partial or full connection mesh.

This particular implementation uses IKEv2 and pre-shared keys. It also employs an always-on VPN tunnel, but feel free to use an on-demand tunnel if it suits your needs better.

## Configuration

- **strongswan-ams3 Droplet configuration**
- **strongswan-nyc3 Droplet configuration**
- **backend-ams3 Droplet configuration**
- **backend-nyc3 Droplet configuration**

## Verification

To verify this setup, we will use Python to run an HTTP server on the backend-nyc3 Droplet. 

```
python3 -m http.server 8000
```

We will then access this HTTP server from backend-ams3 via a private network.

```
curl http://10.108.0.3:8000
```
