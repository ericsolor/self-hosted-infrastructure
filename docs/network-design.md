# Network Design

## Purpose

This document provides a high level overview of the network architecture. It describes how services communicate, how remote access is securely provided, and the methods used to minimize unncessary public exposure while maintaining usability, ease of use, and administrative flexibility.

## Network Topology

The infrastructure follows a centralized architecture with the server acting as the primary service host for the majority of self-hosted applications. All services reside within the internal network and are designed around a VPN-first access model to minimize direct public exposure.

Containerized services are deployed using Docker and communicate through isolated Docker bridge networks or host networking where appropriate. Internal DNS resolution is handled by AdGuard Home, while Caddy provides centralized HTTPS reverse proxy functionality for web-based services.

## Secure Access Design

Remote access is primarily provided through WireGuard VPN. Once connected to the VPN, remote clients become members of a dedicated private network and may securely access internal services through internal DNS resolution and HTTPS reverse proxy endpoints.

### WireGuard VPN Architecture

WireGuard provides secure remote access to internal infrastructure resources. Authorized client devices establish encrypted tunnels to the server, allowing remote users to securely access self-hosted services as if they were locally connected.

The VPN infrastructure is implemented using native WireGuard tooling and manually maintained configuration files rather than third-party management platforms. This approach provides greater visibility into the underlying networking components, facilitate deeper understanding of VPN operation, and retain direct administrative control over peer configuration and routing behavior.

WireGuard was selected due to its lightweight design, modern cryptographic implementation, low overhead, and broad cross-platform support.

### Internal DNS

AdGuard Home is deployed inside a docker container to act as the infrastructure's internal DNS server. It provides centralized internal domain name resolution through DNS rewrites while also blocking advertisements and trackers. This allows internal services to be acessed using hostnames rather than IP addresses; limiting internal service port exposure and allowing HTTPS access.

### Reverse Proxy

Caddy serves as the centralized HTTPS reverse proxy for web-based services throughout the infrastructure. Caddy was selected because it simplifies TLS certificate management, provides straightforward configuration, and has an officially maintained Docker image.

Rather than exposing multiple service ports directly, Caddy routes incoming HTTPS requests to the appropriate internal service based on the requested hostname. This approach centralizes access management, simplifies certificate administration, and reduces direct exposure of backend services.

### Remote Access Workflow

To authorize a device, a WireGuard configuation file is created with a fresh key pair, then the public key is added to the server's WireGuard configuration file. Then inside the WireGuard application on the client device the configuratio file can be used through the "Import Tunnel from File" Option.

Once the configuration file is loaded into the WireGuard application the tunnel can be established granting acess to the server and internal services. Easiest confirmation is to see if data is sent and recieved

![WireGuard Connection Confirmation](../diagrams/ConnectionConfirmation.png)

## Service Exposure Service Strategy

## Docker Networking

## Firewall Interaction

## Network Diagrams

