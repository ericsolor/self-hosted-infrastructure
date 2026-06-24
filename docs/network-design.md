# Network Design

## Purpose

This document provides a high level overview of the network architecture. It describes how services communicate, how remote access is securely provided, and the methods used to minimize unncessary public exposure while maintaining usability, ease of use, and administrative flexibility.

## Network Topology

The infrastructure follows a centralized architecture with the server acting as the primary service host for the all self-hosted applications. All services reside within the internal network and are designed around a VPN-first access model to minimize direct public exposure.

Containerized services are deployed using Docker and communicate through isolated Docker bridge networks or host networking where appropriate. Internal DNS resolution is handled by AdGuard Home, while Caddy provides centralized HTTPS reverse proxy functionality for web-based user interfaces or services.

## Secure Access Design

Remote access is primarily provided through WireGuard VPN. Once connected to the VPN, remote clients become members of a dedicated private network and may securely access internal services through internal DNS resolution and HTTPS reverse proxy endpoints. This essentially gives two layers of protection when accessing internal services, the VPN tunnel is encrypted using [WireGuard's Encrytion] along with HTTPS provided through Caddy. 

### WireGuard VPN Architecture

WireGuard provides secure remote access to the server by port forwarding a nonstandard port to the server. Authorized client devices establish encrypted tunnels to the server, allowing remote users to securely access self-hosted services as if they were locally connected.

The VPN infrastructure is implemented using native WireGuard tooling and manually maintained configuration files rather than third-party management platforms. This approach provides greater visibility into the underlying networking components, facilitate deeper understanding of VPN operation, and retain direct administrative control over peer configuration and routing behavior.

WireGuard was selected due to its lightweight design, modern cryptographic implementation, low overhead, and broad cross-platform support.

### Internal DNS

AdGuard Home is deployed inside a docker container to act as the infrastructure's internal DNS server. It provides centralized internal domain name resolution through DNS rewrites while also blocking advertisements and trackers. This allows internal services to be acessed using hostnames rather than IP addresses; limiting internal service port exposure and allowing HTTPS access.

### Reverse Proxy

Caddy serves as the centralized HTTPS reverse proxy for web-based services throughout the infrastructure also operating within a docker container. Caddy was selected because it simplifies TLS certificate management, provides straightforward configuration, and has an officially maintained Docker image.

Rather than exposing multiple service ports directly, Caddy routes incoming HTTPS requests to the appropriate internal service based on the requested hostname. This approach centralizes access management, simplifies certificate administration, and reduces direct exposure of backend services.

### Remote Access Workflow

To authorize a device, a WireGuard configuation file is created with a fresh key pair, then the public key is added to the server's WireGuard configuration file. Then inside the WireGuard application on the client device the configuratio file can be used through the "Import Tunnel from File" Option.

Once the configuration file is loaded into the WireGuard application the tunnel can be established granting acess to the server and internal services. Easiest confirmation is to see if data is sent and recieved

![WireGuard Connection Confirmation](../diagrams/ConnectionConfirmation.png)

## Service Exposure Strategy

Services are exposed according to the principle of least exposure. Administrative and web-based services are generally accessed through a combination of WireGuard VPN, internal DNS resolution, and HTTPS reverse proxy endpoints provided by Caddy. Direct host port exposure is avoided where possible in order to reduce attack surface and centralize access management.

Certain services may intentionally deviate from this model when operational requirements or client compatibility considerations need alternative access methods. Such exceptions are evaluated on a case-by-case basis and remain restricted through VPN access and firewall controls.

For more information on service exposure view:
- `docker-containers.md`

## Docker Networking

Docker is used to provide service isolation and simplify application deployment. Most services are deployed on Docker bridge networks, allowing containers to communicate with each other internally without requiring every service port to be published directly on the host system.

Where possible, containers are accessed through internal Docker networking and routed through Caddy for HTTPS reverse proxy access. This reduces unnecessary host-level port exposure and allows web-based services to be managed through a more centralized access path.

## Firewall Interaction

The firewall acts as an additional enforcement layer between the host system, local network, VPN clients, and exposed services. Firewall rules are designed to support the VPN-first access model by denying inbound access by default and allowing required traffic for approved services.

UFW is used to manage host-level firewall policy. Access to administrative services is restricted where possible, with trusted access paths prioritized through WireGuard.

The Firewall policy works alongside Docker networking, Caddy, and WireGuard to secure the network of the server. Docker controls container-level connectivity, Caddy centralizes HTTPS service access, WireGuard provides encrypted remote access, and the firewall enforces which traffic is permitted to reach the host and services.

## Network Diagrams

### Network Architecture

![Physical Network Layout](../diagrams/Network%20Architecture.svg)

### Secure Service Access

![Secure Service Access](../diagrams/Secure%20Access.svg)