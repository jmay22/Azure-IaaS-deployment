Lab 02 — Secure 2-Tier Web Application on Azure

Series: Azure Cloud Security Labs
Author: Javen


Overview

This lab builds a classic IaaS (Infrastructure as a Service) architecture — a step up from Lab 01's serverless approach.

With IaaS, Azure owns the physical hardware underneath. Everything above that layer is on me — the OS, the software, patching, networking, and credentials. The attack surface is wider, and the responsibility is greater.

The goal here was to deploy two Virtual Machines inside a Virtual Network, isolate them into separate subnets, and use Network Security Groups to control exactly who can talk to what.


Architecture

Internet → snet-web (vm-web-01) → snet-db (vm-db-01)
                                        ↑
                               No public IP. Internal access only.

The web server sits in a public-facing subnet and accepts traffic from the internet. The database server sits in a private subnet with no public IP — completely unreachable from outside the VNet. The only way in is through the web server.


Resources Deployed

ResourceNameConfigResource Grouprg-lab02-[name]East USVirtual Networkvnet-lab0210.0.0.0/16Web Subnetsnet-web10.0.1.0/24DB Subnetsnet-db10.0.2.0/24Web Servervm-web-01Ubuntu 20.04, Standard_B1s, Public IPDB Servervm-db-01Ubuntu 20.04, Standard_B1s, No Public IP


Key Concepts

VNet (Virtual Network)
A private, isolated network in the cloud where resources live and communicate. Private by default, public by exception — nothing gets in unless you explicitly allow it.

Subnets
Segments inside the VNet that group resources by function and apply different security rules to each zone. Separating the web server and DB server into different subnets means each can have its own access controls. Think of it like separating IT workstations from other device types on a corporate network — same building, different access rules.

NSG (Network Security Group)
Azure's firewall rule set. Controls inbound and outbound traffic. Can be applied at the subnet level — so every resource in that zone inherits the rules automatically — or at the individual VM level, similar to how an EDR tool protects individual endpoints in a hybrid environment.

SSH vs RDP
Both provide remote access to a server. SSH is for Linux machines over port 22 (command line only). RDP is the Windows equivalent over port 3389 (full visual desktop). This lab uses Ubuntu VMs, so SSH throughout.

Jump Host Pattern
The DB server has no public IP — there's no way for external traffic to reach it directly. To access it, you SSH into the web server first, then jump across to the DB server from inside the VNet. One hardened public entry point. Everything sensitive sits behind it. This is a real enterprise security architecture pattern, not just a lab workaround.


NSG Configuration

A custom inbound rule was applied to vm-db-01:

SettingValueSourceIP AddressesSource Range10.0.1.0/24 (web subnet)Destination Port*ActionAllowPriority100NameAllow-Web-Subnet

This ensures only traffic originating from the web subnet can reach the DB server. Everything else is denied.


Security Consideration

Placing a database server in the same subnet as a public-facing web server is a serious misconfiguration. A public-facing subnet is configured to allow external traffic — and that would mean your most sensitive data is reachable from the internet.

Segment your network. Isolate your resources. Control the flow.


Naming Conventions

Consistent naming across every resource makes the architecture readable and speeds up incident response.

ResourceConventionResource Grouprg-lab02-[name]Virtual Networkvnet-lab02Subnetssnet-web, snet-dbVirtual Machinesvm-web-01, vm-db-01


Clean Up

All resources deleted via Resource Group removal post-lab. Orphaned cloud resources expand your attack surface and accumulate cost silently. Always clean up.


LinkedIn Posts (video of the deployment):


[Part 1 — Definitions & Walkthrough](https://www.linkedin.com/posts/javen-m-732a1115a_cloudsecurity-azure-iaas-ugcPost-7472021931421016065-i7aq/?utm_source=share&utm_medium=member_desktop&rcm=ACoAACY8s9YB4E17eGGyOrS3fhvSCDQWNliCmaU)
[Part 2 — SSH Jump & NSG Config](https://www.linkedin.com/posts/javen-m-732a1115a_cloudsecurity-azure-iaas-ugcPost-7472029087826358274-R8qf/?utm_source=share&utm_medium=member_desktop&rcm=ACoAACY8s9YB4E17eGGyOrS3fhvSCDQWNliCmaU))
