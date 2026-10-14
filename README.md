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

## 2. Promote DC01 to a domain controller

I promoted DC01 using the AD DS Configuration Wizard and created a new forest.

| Setting | Value |
|---|---|
| Deployment | Add a new forest |
| Root domain name | `lab.local` |
| Forest / domain functional level | Windows Server 2016 (wizard default) |
| DNS server | Enabled |
| Global Catalog | Enabled |
| NetBIOS domain name | `LAB` |
| DSRM password | Configured |

This is the step that creates the domain and makes DC01 a domain controller. DC01 also runs DNS, which is how the client later finds it.

After the automatic restart, DC01's properties in Server Manager show the domain `lab.local`.

![New forest and root domain name](screenshots/05-new-forest-domain-name.png)

![Domain controller options](screenshots/06-domain-controller-options.png)

![DC01 in the lab.local domain after promotion](screenshots/07-domain-controller-promotion-complete.png)

## 3. Create the OU structure

In Active Directory Users and Computers, I created a top-level `_Branches` OU, a `Houston` OU inside it, and three OUs under Houston for users, workstations, and laptops. I left accidental-deletion protection turned on as I created them.

```
lab.local
└── _Branches
    └── Houston
        ├── Users
        ├── Workstations
        └── Laptops
```
I organized the OU structure by location first, then by object type. That way, a Group Policy could later be applied to all branches, to just Houston, or to only Houston’s users or computers. Group Policy can be linked to OUs but not to the default Users and Computers containers, which is why I built my own OUs.

![Creating the _Branches OU](screenshots/08-create-branches-ou.png)

![Creating the Users OU under Houston](screenshots/09-create-branch-subous.png)

![The finished OU structure](screenshots/10-verify-ou-structure.png)

## 4. Create domain users

I created three user accounts in `_Branches → Houston → Users`:

| Full name | Logon name |
|---|---|
| Alice Johnson | `ajohnson` |
| Bob Martinez | `bmartinez` |
| Chris Walker | `cwalker` |

I put the accounts in the Houston Users OU so that branch-level policy and delegation can apply to them later, instead of leaving them in the default Users container.

![Creating the Alice Johnson account](screenshots/11-create-alice-johnson.png)

![The three users in Houston/Users](screenshots/13-verify-houston-users.png)

## 5. Create security groups

I created a top-level `_Groups` OU to hold the groups, then made three global security groups in it and added one user to each:

| Group | Scope | Type | Member |
|---|---|---|---|
| Helpdesk | Global | Security | `ajohnson` |
| Accounting | Global | Security | `bmartinez` |
| ITSupport | Global | Security | `cwalker` |

The goal is to manage access through groups instead of individual users. Each user is added to a group that matches their role, and allowing permissions to be assigned to the group rather than to individual accounts. I created a dedicated `_Groups` OU to keep groups separate from the branch OUs.

![Creating the Helpdesk group](screenshots/14-create-helpdesk-group.png)

![The three security groups in _Groups](screenshots/15-verify-security-groups.png)

![ITSupport group members](screenshots/16-verify-security-group-membership.png)

## 6. Join Client1 to the domain

I created a second VM, Client1, on the same `adlab` Internal Network as DC01. Before joining the domain, I set its preferred DNS server to DC01 (`192.168.10.10`), since the client uses DNS to find the domain controller.

I then joined Client1 to `lab.local` with a domain administrator account and restarted. Joining creates a computer account for Client1 in Active Directory and sets up a trust relationship between the client and the domain.

After the restart, Client1 showed as a member of `lab.local`, and its computer account appeared in Active Directory. It's in the default Computers container, because that's where a newly joined computer lands until it's moved into an OU.

![Client1 DNS pointing to DC01](screenshots/18-client01-network-configuration.png)

![Client1 in the lab.local domain](screenshots/19-client01-domain-membership.png)

![Client1 computer account in Active Directory](screenshots/20-verify-client01-computer-account.png)

## 7. Test authentication

On Client1, I signed in using the domain account `ajohnson`, with the sign-in domain set to `LAB`. The login worked, confirming Client1 could find DC01 and authenticate the account.

I then ran `whoami /groups` to see the groups attached to the session. The output includes `LAB\Helpdesk`, the group I added `ajohnson` to, along with the default Windows groups. That confirms the Helpdesk group membership I set in Active Directory was applied to the user's logon session.

![Signing in to Client1 as a domain user](screenshots/21-client01-domain-user-login.png)

![whoami /groups output showing LAB\Helpdesk](screenshots/22-client01-verify-helpdesk-group-membership.png)
