<!--# Syllabus
- background on access control in a client/server model
- Kerberos and SSO. Key concepts of Kerberos and overall operation: architecture, operation
- How Kerberos (for SSO), LDAP (for reliable and distributed data store), and DNS (for naming and ressources location) integrates together to make up ADDS networking.
- Windows Domain concept, principal names and types, Domains naming
- Common AD objects and what do they represent: Domain, Organizational Unit (OU), Groups, Users, Computers, Service, Builtin objects (through a demo)
- How initial (the interactive) domain logon is performed (to the workstation), and how subsequent network logon are performed
- Steps taking place when a session is first established: authorizations, SID, Process spawning, script download
- Group Policy: the concept, the GP Object (GPO), aspects that can be controlled via GPOs (Password policies, Audit policies, etc.)
- Other objects in AD: Domain, Domain Tree, Forest, Trust
- Cross domain resources access and Trust. How implemented in AD (using Kerberos), The default trusts,
- Delegation of administration in AD, how implemented in AD
- AD Partitions: Configuration, Schema, GC/PAS, DNS integrated zones, etc. How useful, the replication scope
- FSMO Roles
- AD Replication operation and related objects in AD (Site, ), why and how one should control the replication,
- How to deploy an ADDS environment: windows server roles, functionalities
- How adding Users/Computers to a domain
- closer look at the default resource records created in DNS.
- Common WS administrative tools: "Users and Computers", "Sites and Services", "Domains and Trusts", ADSI Edit, DCPromo, DCDiag, LDP, NetDom, NTDSUtil,
- Misc: Functional levels, advantage of AD-based networking over the legacy Windows Workgroups networking
- Demo: The Domain default GPO, Password policies
-->


# Windows Active Directory vs Windows Workgroups
- In Windows Workgroups, user accounts are local to machine -> users must have a location account in each machine they are suppose to use
  - interactively, during logon
  - non interactively when accessing a file share or any other resource.
- protocols involved: SMB, NetBIOS, WINS, etc.
- Directory enabled networking vs 


# ADDS/DEN Features
- domain wide accounts vs local accounts
- configuration is applied on demand: defined by the admin, located in a configuration store, loaded and applied on local machines dynamicallay
<!--![Active Directory role in an entreprise](../imgs/windows-AD-as-a-hub.png)-->

# Active Directory Domain Services Technical Features
- a system for managing and controlling access to resources in a (local) network
  - achieved using Kerberos, where a domain controller acts as a KDC
  - single sign-on (SSO) to access the domain wide resources. Interactive authentication is done only once (SSO), no need for further interactions with user principals.
  - audited access
- a system for centrally managing an entreprise  configuration (user accounts, desktop environments, apps configuration, permissions, etc.). -> achieves the concepts of DEN
  - achieved using LDAP, used for storing objects describing various aspects of the network componenet and its configuration. the object database is called *active directory*.
    - in prior Windows versions, storage were achieved by Registries and proprietary storage.
  - policy based management, meaning that administrators defines a common configuration for multiple groups of objects, hence the *group policy* name
  - configuring policy and managing active directory objects can be delegated
- resources: files (and network shares), printers, internet access, workstations, etc.
  - controlled through various network services (file servers, proxies, etc.) responsible for enforcing access control
- users principals typically requests access to resources from server principals 
- The AD domain services are:
  - Kerberos, used for domain-wide authentication.
  - Lightweigth directory access protocol, used to manage a central store containing domain components configuration.

- The AD domain services are supported by other services:
  - File replication service (FRS)
  - Domain name service (DNS)
  - Network time protocol service (NTP)
  - and many others. The complete list can be found on the official documentation (especially to make it work across firewalls.)



# Windows AD Domain
- a Windows domain maps to a kerberos realm of security principals (administrative boundary)
- principals are assigned domain identifiers foulen@acme.local (and a SID)
    - domain name forms a naming context for domain elements
- a domain is controlled by a windows server with the ADDS role installed: the domain controller
  - hosts the AD domain services: KDC, NTP, LDAP, etc.
  - authenticates principals and distributes tickets to access domain resources (acts as Kerberos KDC)
  - distributes configuration,
  - performs DNS resolution,
- domain names uses registered DNS domain names (recommendation to ensure uniqueness)
    - a DNS deleguation is not required
- redundant domain controllers

# Principals
- A *security principal*, i.e. User Account, Machine Account, and Group Account is identified by:
  - Security ID (SID), unique across the entire AD objects, assigned at creation time.
  - a name, SPN or UPN, which makes the logon name. It is also possible to have instead of the UPN/SPN, an SAM account name, used for system-local accounts.
- SID is used instead of names in ACLs
- A security principal is represented by an object of type account.
- UPN have the form user@ADDomainName. It is stored in the userPrincipalName attribute name (have nothing to do with the object DN).

# Storage of domain information
- Information controlling the ADDS-enabled network are manyfold: user accounts, policies, groups, application configuration, LAN configuration, DNS records, computers, services, etc.
  - This information is stored in an LDAP directory within mulitple **domain controllers**.

- a directory is made of hierarchically organized objects with different semantics
  - servers, workstations, dc retrieves required objects through the Lightweight directory access protocol (LDAP)

- Microsoft defined classes to accomodate various ADDS-enabled system: User, Computer, OU, Site, etc. Defined in the **AD Schema**.
- The directory schema defines object classes, how they are named, the tree structure rules (what classes can have parent-child relation), data types, access rules, etc.
- a directory may contain multiple domains, w/ or w/o common root naming context.
- replication can be controller on a per-partition basis


# Active Directory Schema
- Defines the classes, attributes, associated structural rules, ACLs, etc. for an AD
- Unique throughout the entire Forest
- Can be defined from within the *Schema Master*
- The directory information schema is stored within the directory itself (in a dedicated partition, Configuration).

# Active Directory Partitions
- AD is divided into **partitions**. An LDAP partition have by definition a seperate naming context and replication rules.
- The AD contains 5 partition types:
  - a per-domain partition containing directory objects of specific domain. As such, they only replicate among the domain controllers of a domain. 
  - "the directory schema" defines classes, attributes, hierarchy for the entire forest
- "Partial attributes set" from all forest domains
- "Configuration" to define forest structure (partitions, references), topology (sites, links, replication links) and specific applications configuration (used for instance in Exchange). Replications throughout the AD
- "Application" to host application related configuration (DNS, Lync, etc.)
- PAS and Schema are replicated througout all the forest.

# Domain controllers roles
- Domain controllers are basically Windows Servers equipped with ADDS feature; Moreover, they are assigned with one or more roles depending on the active directory partitions they hold and *operation* they are responsible for.
- domain controllers: domain + schema + config + app
- global catalogs (gc): pas + schema + config
  - used to speed up search operations and reduce traffic across the forest
  - updated on domain data changes
  - available in the first controller in the domain
- read-only domain controllers (RODC): used in sites where having a lower security level of the domain controllers (e.g. adhoc or remote deployments)
- Operations masters: Some operations needs a single master domain controller:
  - Schema Master, where schema can be modified
  - Domain Naming Master, for issuing names
  - RID Master, for assiging identifiers
  - PDC Emulator, for backward compatibility with legacy Windows NT Domains.
  - Infrastructure Master, for controlling certains objects (Sites,)
  Windows refers to the separation of the DC Operations master as *Flexible Single-Master Operation* (FSMO).

# Access control model
- Security principals include users and groups. Security principals perform actions on objects, which include files, folders, printers, registry keys, Active Directory directory service objects, and other types of objects. Security principals sont les objets à qui des permissions peuvent être accordées sur les ressources. Chaque security principal est identifié par un SID attribué lors de sa creation. Ceci est fait de deux partie, domain id et RID.
- Each object has an owner that grants permissions to security principals. During the access control check, these permissions are examined to determine which security principals can access the object and how they can access it.
- A thread assumes the rights and privileges of the security principal that initiated the process.
- When a user logs on, the system creates an access token for that user. The access token contains the user’s SID, the SIDs for any groups the user belongs to, and the user’s privileges. This token provides the security context for whatever actions the user executes on that computer.
- Before allowing a subject to proceed with the action it intends to carry out on an object, the operating system’s security subsystem performs an access check to determine whether the subject is authorized to perform the action. The access check systematically compares information in two key data structures:
  - The subject’s access token, which contains a SID for the user and additional SIDs for groups that the user belongs to.
  - The object’s security descriptor, which contains a discretionary access control list (DACL) with a list of access control entries (ACEs) that specify the access rights that are allowed or denied to particular users or groups, each of whom is identified by a SID.

- the security descriptor defines protection and audit rules related to the resource. It contains an the owner, primary group, an DACL and SACL
<!-- ![Access Control Model](../imgs/authorization-and-access-control-process.png) -->

# How access control to resources is implemented
- Security principals are issued SID at accounts Creation
- Resources are assigned ACLs which defines access permissions
- On Authentication, the user SID and its groups SID are fetched
- On resource access the server enforces the access control
- Security concepts: SID, ACLs/ACEs

- How initial domain logon (interactive) is done:
  - the various Kerberos authentications which take place
- Cross-domains resources access
  - How Kerberos is being used and the concepts of


# How initial authentication happens?
- user performs interactive authentication when accessing to *domain-joined* workstation
  - done only once (sso), domain logon
- interactive authentication uses either passwords, smart cards, may be an MFA
- users is the client principal, workstation contains the server, and the KDC 
- authentication steps:
  - find a controller for the user domain. uses special DNS records.
  - the client is redirected to his domain controller and performs kerberos authentication
  - the clients gets a ticket for the local workstation
  - a user session is started; created user processes uses the user identity/list of groups it bleongs to, and desktop environment configured using adequate GP objects

# Subsequent resources access
- workstation caches user tickets
- resources are assigned ACLs which defines access permissions
- on Authentication, the user SID and its groups SID are fetched
- on resource access the server enforces the access control

# Access across domains
- Authentication is done by the home domain. Principals gains access to resources in foreign domains through the trust between domains.
- Access permissions are at the discretion of the foreign domain administrator

# Policies-based administration
- Group policy (using GPO)
- Audit Policy
- Password Policies


# Forest
- a collection of domains, typically belonging to an enterprise
- what forest domains have in common:
  - (mutual) trust between domains can be established
  - AD schema
  - configuration partition
- sso applies across the the forest:
  - authentication is performed in the home domain
  - remote domain controllers delivers tickets for the remote servers
- domain controllers can mutually or unilaterally trust authentication performed within the forest

# Organizational Units
- an AD object used as a container to group objects in a domain for various purposes:
  - apply the same group policy on multiple objects
  - delegate management to specific administrators (implemented behind the scene with ACLs on the OU objects)
- different from Generic Container classes (like the `cn=Computers`)
- defined within a domain, can contain any users, computers, groups, or other organization units



# Group Policy
- A Group Policy (GP) allows administrator to implement policy based management.
- The GPO contains the following settings (more than 2500 in W2K8):
  - Security: defines account policies (Password Policy, Account Lockout Policy,), Local Policies (Audit Policy, User Rights Assignment), IPsec policies, etc.
  - Scripts: defines the scripts run at computer session startup/shutdown, user session logon/logoff
  - Preferences: defines global Windows settings: Control Panel, environnements vars, available shares/Folder Redirections, etc.
  - Software installation
  - Administrative templates
  - etc.

- GPOs can be linked to containers objects like OU, Site ou Domain, or to local system.
- GPOs can apply to a subset of objects of a container by using a `Security Group`
- An object will have muliple, overlaying GPOs which apply: Local GPOS, Site GPOs, Domain GPOs, OU GPOs, traités dans cet ordre.
- Creating a GPO requires two steps:
  - create the GPO within the domain
  - then link to any container within the domain (or the domain itself.)
- A GPO have two categories of settings: those which apply to user session and those wich apply to computer session.
- We typically define a GPO for Computers and another one for Users.
- Each domain has a default domain GPO
- Windows Server provides two Management console snap-ins to hangle GPOs: GPME and GPMC

# Group Acccounts
- AD defines a *group account* (aka *security group*), which help is some scenarii:
  - to be used as a wlidcard reference when defining permissions to a group of accounts (computers or users)
  - in deleagation
- There is another type of groups called *distribution group*, used in mail (to be used as receipients)
- Some predefined groups, like `Domain Admins`.



# System auditing


# Administration delegation
- within each domain, a specific group of users are given administrative rigths on the domain objects, these are the domain admins, stored in the group `Domain Admin`
- Some admins can be restricted access to some OUs, Sites or Domains.

# Sites How they are used in Replication
- A container like object used to group `Domain Controllers` objects
- `Domain Controllers` within a LAN are typically grouped in a Site. What do they have in common:
  - high speed links, low cost communication
- Sites configuration allows admins to
  - control replication frequency: make replication within a site happen more frequently than across sites
 (to accomodate cost, bandwidth contraints or peak traffic hours)
 - define the preferred domain controllers for clients to used during authentication or various queries
- When a client wants to authenticate, it defaults to using the domain controller in its own site. This is achieved in the `SRV` record during the domain controller name resolution.


# Naming of AD objects
- Depnding on the view point, an AD object is identified with:
  - Globally Unique Identifier (GUID), immutable, used to distinguish objects even among old, removed ones. It is stored as an object attribute
  - Security Id (SID), used for objects representing principals, and is typically referred to in authorization ACLs
  - Distinguished name (DN), exported by LDAP
    - Example `dn:cn=Mounir,ou=Users,ou=Marketing,dc=entreprise,dc=com`
  - Canonical name: Windows Server tools uses this instead of `dn` to refer to AD objects. Non Windows tools might still use dn notation.
    - Example: `entreprise.com/Marketing/Users/Mounir`
  - LDAP URL
    - Example: `ldap://unserveurLDAPduDomaine/cn=Mounir,ou=Users,ou=Marketing,dc=entreprise,dc=com`

# Setting up an AD Domain using PS cmdlet
  - Add the role
    - will also add the ADDS/ADLDS administration tools (GUI and command line like dcdia.exe)
  - Install-windowsfeature -name AD-Domain-Services -IncludeManagementTools
  - Get-Command -Module ADDSDeployment
  - Get-Help <cmdlet name>
  - Issue the command
```
  Install-ADDSForest
      -CreateDnsDelegation:$false
      -DatabasePath C:\Windows\NTDS"
      -DomainMode "Win2012R2
      -DomainName "yourdomain.com"
      -DomainNetbiosName "YOURDOMAIN"
      -ForestMode "Win2012R2"
      -InstallDns:$true
      -LogPath "C:\Windows\NTDS"
      -NoRebootOnCompletion:$false
      -SysvolPath "C:\Windows\SYSVOL"
      -Force:$true
```
  - You will be asked for the domain Administrator password and the restore mode password
<!-- Slide: Security concepts: SID, ACLs/ACEs

# Some LDAP commands 
- ldifde
- ldp
- dsuery (requête ldap)
- ldifde
```bash
$ldifde -i -f theFileBelow.ldif -d dc=domainename
DN: CN=SampleUser,DC=DomainName
changetype: add
CN: SampleUser
description: DescriptionOfFile
objectClass: User
sAMAccountName: SampleUser
```
 -->

# Further reading
- Microsoft Technet is a good reference and contains good explanation of most of the ADDS internals.
<!-- - https://www.serverbrain.org/active-directory-2008/kerberos-authentication-protocol.html
- a good explanation on LDAP vs RDBMS (firstanswer) https://stackoverflow.com/a/2293560/660035 -->

