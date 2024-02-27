# DigitalOcean-Site-to-Site-VPN

Site-to-site VPNs help privately connect your on-premises environments with your cloud environments on DigitalOcean. They also facilitate secure inter-VPC communication, known as VPC peering. This guide demonstrates how to establish a site-to-site IPsec VPN with StrongSwan using a Droplet as a VPN gateway.

## Architecture

Let's imagine you run workloads in two DigitalOcean data centers in different regions and these workloads need to communicate privately. The workloads run on the backend Droplets that we will privately connect together over a VPN tunnel running between a pair of gateway Droplets, as you can see from the diagram below.
![Architecture Diagram](https://lucid.app/publicSegments/view/24b18af7-7e06-4d37-b7f8-d117aae9f0e3/image.png)

## Configuration

## Verification
