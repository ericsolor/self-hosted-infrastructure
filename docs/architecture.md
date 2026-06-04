# System Architecture

## System Purpose

This system is built to perform as a secure, modular self hosted infrastructure for a variety of services. Its initial conception was for NAS purposes; significantly enhancing my filesystem accessibility without the need of cloud storage. 

During my 2nd year in college I took an introductory physics course that I thoroughly enjoyed. Throughout the semester, I used an iPad and Apple Pencil to annotate lecture slides directly and stored all of my notes in iCloud under the assumption that they would remain safely backed up. Approximately one week before the final exam, I opened my notes to begin studying and discovered that the folder containing an entire semester's worth of handwritten lecture notes was completely empty. The files vanished as if they never existed. Despite numerous recovery attempts, the notes could not be recovered. While researching the issue, I discovered many reports from other users describing similar experiences involving missing or unexpectedly deleted files within cloud storage platforms.

That experience served as a lesson in the risks associated with placing critical data entirely under the control of third-party services. It highlighted the tradeoff between convenience and ownership, and ultimately became the catalyst for building my own self-hosted storage solution. 

As development progressed, the scope began to evolve beyond its original NAS-focused roots. What began as a file storage platform gradually expanded into a broader self-hosted infrastructure environment used for experimentation, systems administration, networking, cybersecurity, containerization, AI workloads, and continuous technical learning. Today, the system serves not only as a storage platform, but also as a practical environment for implementing new technologies, validating architectural concepts, and reducing dependence on external services whenever appropriate. 

## Hardware

### Core internals

- MSI PRO Z690-A DDR4 ProSeries Motherboard
- Intel Core i7 12700KF
- G.SKILL AEGIS Series (Intel XMP) DDR4 RAM 16GB (2x8GB) 3000MT/s
- CORSAIR RM750e Fully Modular ATX Power Supply, Gold Efficiency, 105°C-Rated Capacitors

### Graphics

- ASUS GeForce RTX™ 5070 Ti OC Edition 16GB

### Storage

- Seagate IronWolf 8TB NAS Internal Hard Drive
- SAMSUNG 870 EVO SATA SSD 500GB 2.5” Internal Solid State Drive (x2)

## Core Technologies

### Operating System

This system was built around Debian because of Debian's reputation for stability, predictability, massive package ecosystem, and minimal overhead. Currently, this system runs Debian Trixie, rather than Bookworm, due to the NVIDIA Blackwell drivers needed for the 5070ti not being compatible with older kernels used in Debian Bookworm. Additionally, upon installing the OS, no window manager or desktop environment was installed because this system is intended to function primarily as a headless infrastructure server; therefore, graphical desktop environments would introduce additional resource consumption, maintenance overhead and a potential attack surface with little to no added benefit. 

Other operating systems were considered during the initial deployment of this system such as: TrueNAS, Unraid, and Ubuntu. While TrueNAS and Unraid offered simplicity and ease of use, they did not provide the same flexibility, direct low level system control, and room for experimentation as a standard Debian deployment. Additionally, Debian was chosen over Ubuntu due to Debian being a cleaner and more minimal foundation.

### Containerization and Service Management 

Docker Compose is used throughout this system to simplify the deployment of services as well as separate dependencies from the system host. This helps avoid depenancy conflicts, maintain a cleaner host system, and limit the potential impact of a compromised or misconigured service. Additionally, a containerized approach allows services to be modular, simplifies service management, and streamlines recovery.

**TECHNOLOGIES CURRENTLY DEPLOYED:**
- Docker Engine
- Docker Compose
- Docker bridge networking

For more information regarding containerization infrastructure design, and implementation, see:
- `docker-containers.md`

### Security and Access

Networking and remote accessibility throughout this system is designed to minimize public exposure while still providing secure remote access. Wireguard was selected as the primary remote-access solution due to it's lightweight desing, strong modern cyptography, low overhead, and cross platform support. Caddy was chosen to provide centralized HTTPS reverse-proxy routing for internally hosted web services because it simplifies TLS certificate management, limits direct service exposure, and already has an official Docker image. This approach allows web-based administrative interfaces and self-hosted services to remain securely accessible without exposing large portions of the internal infrastructure directly to the public internet. AdGuard complements caddy as a local DNS manager through DNS rewrites and internal domain resolution. Adguard was selected because its lightweight deployment, web-based management interface, and an official Docker image.

**TECHNOLOGIES CURRENTLY DEPLOYED:**
- UFW Firewall
- Wireguard VPN
- Caddy Reverse Proxy
- Adguard DNS

For additional information regarding networking architecture, firewall strategy, and credential handling, see:
- `network-design.md`
- `firewall-strategy.md`
- `secrets-management.md`

## Thermals and Cooling

This system is housed in a standard-size computer tower, primarily built for ATX motherboards. There are three 120mm fans installed; two are installed at the top of the tower and one is installed on the back of the tower. On the top of the tower closest to the front, the fan is configured to act as an intake fan to bring in and circulate cooler ambient air throughout the chassis. Next to it, the fan located at the top of the tower closest to the back is directely above the CPU, pulling out rising hot air generated by the CPU cooler. The rear exhaust fan follows the same principle and assists in pulling hot air out of the chassis. 

In addiation, this system utilizes an ARTIC Liquid Freezer III AIO CPU cooler with a front mounted radiator configured as an intake. With the radiator pulling in cooler ambient air, the CPU cooler can cool the CPU with greater efficiency and maintain lower operating temperatures while further assisting airflow circulation throughout the chassis. This configuration is especially important because the 5070ti has an open air axial fan design which pulls air in from beneath the GPU and dispurses generated heat back into the center of the chassis. The resulting airflow pattern helps reduce CPU temperatures while maintaining continuous movement of warm air towards the rear and top-rear exhaust fans.

### AVERAGE TEMPS:

### DIAGRAM:

![Air Flow Diagram](../diagrams/AirFlowDiagram.png)

## Infrastructure Philosophy

These principles guide architectural decisions throughout the infrastructure. Services are designed to remain modular and independently manageable, maintenance tasks should be straightforward and repeatable, resources should be utilized efficiently, and security should be incorporated into the architecture rather than treated as an afterthought:

- Modularity
- Maintainability
- Efficiency
- Security 

Security throughout the infrastructure follows a layered defense strategy focused on minimizing unnecessary exposure, reducing attack surface, and maintaining manageable operational complexity. Rather than exposing services directly to the public internet, administrative access is designed around a VPN-first model. Services are deployed with the assumption that failures, misconfigurations, and compromises can occur; therefore, architectural decisions attempt to limit the potential impact of any single service becoming unavailable or compromised. This philosophy and many architectural decisions are based on the principles established by the CIA Triad. 

### Confidentiality

    - Raw Wireguard VPN access
    - Reverese-Proxy HTTPS access to web UI's
    - Hardened SSH configuration
    - Explicit firewall rules for exposed ports and IP ranges
    - Limited public exposure for administrative services
    - Non-standard ports
    - Rule based access restrictions

### Integrity

    - Containerized service seperation and Docker Compose
    - Read-only volume mounts where appropriate
    - Backup and redundancy planning
    - Environment variable separation for sensitive configuration data
    - Modular infrastrucutre organization and configuration management

### Availability

    - Lightweight headless Debian OS to to reduce unnecessary overhead
    - Thermal and airflow optimization for sustained operation
    - Modular containerized services to simplify maintenance and recovery
    - Segmented infrastructure design to reduce single points of failure
    - Automated service and system health checks
    - VPN based remote accessibility

### Design Constraints

This system is designed to operate with a limited number of users for reasons such as: limited recourses, physical space, residential network bandwidth, temperature regulation and electricity draw. Therefore, this system prioritizes practicality and simplicitly where appropriate to avoid unneceassry complexity while still allowing room for future scalability and experimentation.

## Limitations and Concerns

### RAID Array

This system does not currently implement a RAID array which is unusual for its original intended purpose and its current scope. Given the cost of NAS-grade storage components, resources were instead allocated toward building a functional and expandable infrastructure platform. To compensate for the absence of RAID, alternative backup and redundancy mechanisms were implemented to protect critical data and simplify recovery procedures.

As the infrastructure continues to mature, RAID-based storage redundancy remains a planned future enhancement. 

For more information on the current back and redundancy strategy, see:

- `backups-and-redundancy.md`

### Drive Airflow Hotzone

Currently there is no fan dedicated to assisting the airflow circulation of the storage drives. This is a notable concern, however, current temperature monitoring assures that the drives are operating at optimal temperatures. Nevertheless, continuous monitoring remains important, as sustained high-I/O workloads, elevated ambient temperatures, or future hardware additions could alter thermal characteristics within the chassis and impact long-term drive reliability.

### Single Host Infrastructure

The majority of the infrastructure services currently reside on a single physical host creating a single point of failure. Containerization and backup planning reduce risk, however, hardware failure affecting the host could impact multiple services simultaneously. Future infrastructure expansion could include multiple nodes to reduce single points of failure. 

### Residential Network Dependency

This infrastructure operates on a residential network subject to heavily limited upload bandwidth. As a result, external accessibility, bandwidth and uptime are dependent on the reliabiliity of the ISP and local power availability. While this is acceptable for the current scale, for any future large scale deployments this becomes a problem. 

### Limited Hardware Resources

While in certain areas the hardware is more than capable for current workloads, there are certain areas which raise concern. Primarily only having 16GB of DDR4 system RAM at 3000 MT/s. This concern is mitigated through carefult consideration of background processes, concurrent running services, AI workloads and startup sequences. As development continues this will increasingly become a limiting factor. 

### Ongoing Development

This self hosted infratstructure is continuosly evolving. It should be looked at as an ongoing project with no defined end. As new technologies, requirements and opportunities emerge, the infrastrucute will continue to adapt. New services are regularly evaluated, architectural decisions are refined, and existing implementations are improved as new knowledge and experience are acquired.