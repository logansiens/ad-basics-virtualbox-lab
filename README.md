# Active Directory Home Lab (VirtualBox)
A small Active Directory environment built on my own PC with two Windows Server 2022 virtual machines in VirtualBox: a domain controller (DC01) and a domain-joined client (Client1). I created the domain, built an OU structure, added users and security groups, joined the client, and tested domain authentication.

## What this project demonstrates
- Installing the AD DS role and promoting a server to a domain controller (new forest, DNS, Global Catalog)
- Building a location-based OU structure to support Group Policy and delegation
- Creating domain users and role-based global security groups
- Configuring DNS so a client can find the domain controller
- Joining a computer to a domain and verifying authentication and group membership
- Building and troubleshooting a two-VM lab network in VirtualBox

## Lab environment

| VM | Role | OS | Resources | Network |
|---|---|---|---|---|
| DC01 | Domain controller, DNS | Windows Server 2022 | 2 vCPU, 4 GB RAM, 60 GB dynamic disk | NAT + Internal Network `adlab` (192.168.10.10) |
| Client1 | Domain-joined client | Windows Server 2022 | 2 vCPU, 4 GB RAM, 60 GB dynamic disk | Internal Network `adlab`, DNS set to DC01 |

**DC01**

![DC01 VirtualBox details](screenshots/02-dc01-vm-details.png)

**Client1**

![Client1 VirtualBox details](screenshots/17-client1-vm-settings.png)

### Network

DC01 has two adapters: one on NAT for internet access, and one on a private Internal Network (`adlab`) shared only by the lab VMs. Client1 is on the same `adlab` network so it can reach the domain controller. The lab network uses `192.168.10.0/24`, and DC01 has the static IP `192.168.10.10` with no default gateway, since this network only connects the lab VMs.

## 1. Install the AD DS role

I added the Active Directory Domain Services role through Server Manager on DC01, accepted the required management tools, and left everything else at the defaults.

This installs the AD software but doesn't create the domain yet. The domain isn't created until the server is promoted to a domain controller in the next step.

![Selecting AD DS in the Add Roles and Features wizard](screenshots/04-add-ad-ds-role.png)
