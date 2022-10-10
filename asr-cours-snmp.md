# Overview
- Monitoring is done for various purposes:
	- do I have to upgrade hardware (memory, bandwidth, networking devices, etc.)?
	- timely detect interfaces going down, a system not responding
	- collecting performance data samples for capacity planning, peering planning, DDoS protection
	- early detection of increased/unusual bandwidth consumption
	- build a standard activity/performance baseline for later comparison
	- monitor web sites availability used by end users,
	- detect suspicious activities (data leak by malware, encryption by ransomware, etc.)
- A generic monitoring architecture:
	- **agents** in monitored devices, send sampled data, summary reports or events to a designated **manager** (hosting worskstation is called **Network management station**, NMS), using an application protocol like SNMP, TR-69: either in push or in pull modes.
	- agents and manager have a common **data model** which defines which data may be requested, identifiers, encoding, semantics, etc.

# Best practices in network monitoring
- always establish a baseline profile for the behaviours of interest (traffic composition, server perf, throughput, errors, etc.)
- implement consistent configuration management, allowing you to restore/apply configuration profiles in case a situation happened
- define and follow an escalation matrix defining who is responsible for what
- make monitor at each layer/function in the network (devices, system, runtime engines, app servers, application, web server, etc.)
- implement HA and failover for the monitoring infrastructure itself
- make attention to the licence limits in terms of the number of supported devices/services

# How to set up monitoring using SNMP
- activate the feature on the device
- define your objects and OIDs
- create scripts using GET, tune the poll interval
- the issue is with timestamps, transport reliability

# Simple Network Management Protocol Architecture
- Simple Network Management Protocol (SNMP) is one of the most used protocols in industry for monitoring. Why? Because it is simple to understand, easily implemented, exenstible data model, early IETF standard, wide adoption by OEM/software editors.
- SNMP is built on agents and manager as defined above.
- The protocol offers three use cases:
	- manager send a `Get` request to read the value of an object from the agent's MIB. Other variations of the request include `GetNext` and `GetBulk`.
	- manager send a `Set` request to set the value of an object on the agent's MIB. This is a less common use case (configuration is generally done through other means).
	- upon specific conditions, agents sends a `Trap` request to a preconfigured manager, along with information (object, threshods, the actual value). `Inform` request is an improved version.
- `Inform` requires acknowelegment from manager. this may cause storms in case of mass failures within an administrative domain.
- SNMP uses udp/162 (on NMS for receiving traps) and udp/161 (on agents, for Get and Set). When using TLS or DTLS as transport, it uses 10162 and 10161, respectively.
- SNMP is generally embedded in a user friendly, GUI based monitoring software. Commands implementing Get and Set are also integrated in scripts with specific purposes. Nagios XI, for example, depends on SNMP (among others) to detect services/interfaces availability.

# The Management information base
- The MIB is collection of **management objects** (MO) representing informations or events and implemented by an agent on which it is able to respond to the managers.
- To allow for interoperability among agents and devices, all the SNMP-supported MOs are organized in a single universal hierarchy,
	- creating a single namespace with an hierarchical identificatin scheme
	- the namespace is extensible, where interested organizations can register a dedicated namespace then define whatever objects their devices needs
	- universal namespace allows collision free OIDs, extensible
- Each management object represents a
	- performance metric ()
	- state parameter (sysUpTime, ipAddressTable)
	- configuration parameter
- Management object identification follows an hierarchical schema. For example for the `sysUpTime`,
	- its Object ID is `.iso.org.dod.internet.mgmt.mib-2.system.sysUpTime.`
	- in numerical notation, this becomes `.1.3.6.1.2.1.1.3.`
	- Identifier is derived by following downward direction from the namespace root.
- MIB is defined in a language called **Structure of management information** (SMI)
- Organizations define their MO for their devices and releases them in modules and makes them available to managers to be able to issue requests and decode traps. The same module is supported (typically hardcoded) within the agents.
- There are the following module types:
	- standard modules, located in `.iso.org.dod.internet.management.`, like `mib-2` `.1.3.6.1.2.1.`
	- experimentation modules
	- enterprise modules like `CISCO-PRIVATE-VLAN-MIB` in `.iso.org.dod.internet.private.enterprise.cisco`
- Some enterprise does not publish their module and embeds them into their proprietary management software
- IETF defined `mib-2` module for objects common to all TCP/IP stack: This module is made of the following groups: system, tcp, udp, ip, snmp, etc.
- Terminology note:
	- MIB is used to refer to various things: text files of modules definitions, information an agent holds, universal tree
- you can explore the MIB content of a device using a MIB Browser

![Structure of the MIB universal tree](../imgs/MIB-structure-1bis.png)

# The Structure of Management Information (SMI)
- the schema of the MIB is defined in a language expressed with Abstract Syntax Notation (ASN.1), called Structure of Management informatin (SMI).
- Each object has at least the following attributes:
	- `SYNTAX`: `IPAddress`, `Counter` (an increasing scalar value, e.g. number of packets, ), `Gauge` (a fixed width, variable data, e.g. CPU usage,), `Sequence` (C struct-like data structure), `Sequence Of` (to describe a table of), `TimeTicks`, etc. Other predefined types exists.
	- `MAX-ACCESS`: ro, rw
	- `STATUS`: obseleted, current, optional, etc.
	- `DESCRIPTION`
	- `OID`
- In some contexts, the management information schema and the actual agent database are both referred to as MIBs.
- While the MIB schema is hardcoded into the agent, a manager needs the MIB schema in order to be able to decode and interpret the data sent by the agents, map numeric and symbolic notations, and allow the sysadmin to discover available information and their semantics. Each manufacturer may provide the different MIB their equipements support.
- Following is an exampe of an MO
```
sysName OBJECT-TYPE
SYNTAX  DisplayString (SIZE (0..255))
MAX-ACCESS  read-write
STATUS  mandatory
DESCRIPTION 
    "An administratively-assigned name for this managed node.  By convention, this is the node's fully-qualified domain name."
::= { system 5 }
```
- Objects of type `Sequence Of` defines an index (or multi-indexes) to allow direct access into the table. Here is an example of an index definition:
```
ifTable OBJECT-TYPE
    SYNTAX      SEQUENCE OF IfEntry
    MAX-ACCESS  not-accessible
    STATUS      current
    DESCRIPTION
            "A list of interface entries. The number of entries is given by the value of ifNumber."
    ::= { interfaces 2 }

ifEntry OBJECT-TYPE
    SYNTAX      IfEntry
    MAX-ACCESS  not-accessible
    STATUS      current
   DESCRIPTION
            "An entry containing management information applicable to a particular interface."
    INDEX   { ifIndex }
    ::= { ifTable 1 }

IfEntry ::=     SEQUENCE {
        ifIndex                 InterfaceIndex,
        ifDescr                 DisplayString,
        ifType                  IANAifType,
        ifMtu                   Integer32,
        ifSpeed                 Gauge32,
        ifPhysAddress           PhysAddress,
        ifAdminStatus           INTEGER,
        ifOperStatus            INTEGER,
        ifLastChange            TimeTicks,
        ifInOctets              Counter32,
.... truncated .. 
        ifSpecific              OBJECT IDENTIFIER -- deprecated
    }
ifIndex OBJECT-TYPE
    SYNTAX      InterfaceIndex
    MAX-ACCESS  read-only
    STATUS      current
    DESCRIPTION
            "A unique value, greater than zero, for each interface.  It is recommended that values are assigned contiguously starting from 1.  The value for each interface sub-layer must remain constant at least from one re-initialization of the entity's network management system to the next re-initialization."
    ::= { ifEntry 1 }
```
- The graphical representation of this `Sequence Of` is as follows: 

![mib-2.ifTable Table](../imgs/snmp-ifTable-tableau.png)

- The index of this table is `ifTable.Colonne.ValeurDesIndex`. For example, In order to read the `ifDescr` attribute of interface having `ifIndex` 1, one should read object `ifTable.2.1` 
- Without an index, one should use `GetNext` or `GetBulk` to retrieve the entire table.

# SNMP Packets encoding
- SNMP uses ASN.1 to represent information and BER to enode information into the packet
- BER is a binary encoding, self describing, following a schema TLV (Type, Length, Value). Type tells the type of the information (how it us layed out), on 1 or 2 bytes.
- SNMP uses universal types as INTEGER/OCTET STRING (available form ASN.1), and others, complex (Gauge32, Counter32 IpAddress, etc.) which are defined using ASN.1 basic types.
- Requests/Responses contains a set of VarBindings telling the object ID and its value in BER encoding



# Traps and Informs
- an SNMP agent sends an messages to the manager whenever a preconfigured condition happens.
- SNMP agent uses `Trap` message. Strating form version 2, SNMP introduced `InformRequest`, which is an acknowledged version of Trap, enhancing protocol reliability.
- The MIB includes an MO for each condition of interest. Here is an example:
```
warmStart NOTIFICATION-TYPE
STATUS  current
DESCRIPTION
       "A warmStart trap signifies that the SNMP entity, supporting a notification originator application, is reinitializing itself such that its configuration is unaltered."
::= { snmpTraps 2 }
```

- IETF have defined in the mib2 module some standard conditions:
	- coldStart (0): agent rebootd with data loss
	- warmStart (1): agent rebooted without data loss
	- linkUp (2), linkDown (3): includes the identifier of the interface
	- authenticationFailure (4): an SNMP (get ou set) on the agent undergone a bad authentication
- A condition occurred, the agent must include in it the following information:
	- `sysUpTime` when the trap occurred
	- `snmpTrapOID`, the trap OID
	- list of MO specified in the definition of the Trap.
- Note that conditions are hardcoded in agent and cannot be expressed in the MIB.
Here is another example:
```
cmnMacThresholdExceedNotif NOTIFICATION-TYPE
	OBJECTS {
		cmnUtilizationUtilization,
		cmnMACThresholdLimit,
		cmnUtilizationTimeStamp	}
	STATUS current
	DESCRIPTION "cmnMacThresholdExceedNotif is sent when cmnUtilizationUtilization exceeds or equals to the cmnMACThresholdLimit for a given entPhysicalIndex. cmnMacThresholdExceedNotif is not sent when cmnMACThresholdLimit is set to zero"
	::= { cmnMIBNotifications 3 } 
```
- Most network management software allows sysadmins to define reactions to various traps: sending an SMS or e-mail, makes a visual or alerting sounds, react with a reconfiguration script, simply making a log in the journal, etc.

# SNMP Security
- Major threats to SNMP traffic: non allowed access (Set, Get), traffic interception for sniffing or alteration.
	- is sniffingo a concern in a switched network? what if we only disable SET,
- Only version 3 supports cryptography-based security (authentication, encryption)
- SNMP version 3 offers User-based security model, USM) offering three levels:
	- without security, for backward compatibility with SNMPv1/2c agents 
	- authenticated access
	- authenticated access and encryption (using DTLS/SSH transport)
- In version 2, clients sends clear-text passwords in transactions! The password is called `community` name --> dangerous.
- Role-based and view based access control (VACM) is implemented in agents
- VACM: for each group of community/ip define the allowed views for read, write, notify. views may also specify views within tables
- System administrators typically disable accepting Set in agents for version 2,
- Management LAN can also be used to restrict visibility of the SNMP traffic (even for the Get)

# Remote monitoring
- In cases where the system to monitor is deployed in remote locations, it is possible to have a probe to act as an SNMP NMS and read informations from local agents, store it, making summaries/aggregation/samping and storing it locally. The SNMP defined a RMON-MIB to hold such data and make it available for agent on central NMS stations.

# MIB-2 Content
- 1.3.6.1.2.1.
- system group: mib-2.1: sysDescr, sysObjectID (an id of the private mibs of the agent), sysUpTime, sysContact, sysName, sysLocation, sysServices,
- mib-2.interface: ifNumber, ifTable: Sequence of ifEntry
	- ifEntry: Sequence of (ifIndex, ifDescr, ifType, ifMTU, ifSpeed, ifPhyAddress, )
- mib-2.at (for arp cache), deprecated
- mib-2.ip: ipForwarding, 
- mib-2.icmp
- mib-2.tcp
- mib-2.udp
- mib-2.snmp (11)


# YANG/NetConf/RESTConf
- YANG is a data modelling language to describe mgmt information on a device (equivalent to MIBs)
- NETCONF (NETwork CONFiguration) is a protocol defined by the IETF to “install, manipulate, and delete the configuration of network devices”.
  - NETCONF operations are performed via a RPC layer using XML based encoding
  - key features: ability to rollback configurations, separation of config from operational state
- YANG models both data and operations
	- operations: shutdown, restart, set date and time
- need for transactions, rollback, low implementation costs, and the ability to save and restore the device's configuration data
- Some analogy can be built between YANG/YANG Modules/XML/NetConf and SMIv2/MIB Modules/ASN.1 BER/SNMP.
- it is possible to create streams from commands output (cf. )

# WEBM and DMTF Effort, WMI
- WEBM is a system standardized in order to enable IT systems monitoring and management.
- Made up of
	- CIM -Common information model) for the data model (several UML diagrams to describe the monitored data objects)
	- transport protocols based on HTTP services (WS-Management, WS-RS, etc.). RPC is used by Microsoft in their WMI
- Windows Management Instrumentation (WMI) is the DMTF WEBM implementation
- uses transport by DCOM, which in turn in sbased on RPC (transport in the WebM)
- CIM classes for the managed objects
- architecture in ![WMI Architecture](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-wmi/ms-wmi_files/image001.png)

# Misc notes
- OpenTelemetry
- `Net-SNMP` package provides composite commands:
	- `SNMP walk` goes through an entire hierarchy in a depth-first fashion.
	- `SNMP Table` to read tables and display them with a table layout.
- Get-Next follows a depth-first traversal. 
- Operations Support system: support management functions such as network inventory, service provisioning, network configuration and fault management
- Business support systems: support various end-to-end telecommunication services
- “observability” measures how well we can understand the internals of a given system using only its external outputs (we can’t understand a complex system if it’s a black box)
- Always think about compatibility with currently deployed nodes, for instance: does SNMPv2 NMS can communicate with older agents ? 
- ipfix, netflow, xflow, etc. used mainly for traffic/flows monitoring/sampling

# Assignments
- How to make your device snmp-capable?
- When setting up a client, you should provide it with the MIBs ever device supports. Why? Do we need to make the same thing with the agents?

# Reading
- https://www.tek-tools.com/network/network-monitoring-guide-and-tools#reporting-at-each-layer

---
# Demo
- sending trap under a link down condition or reboot condition
- polling snmp agent to make local charts using scripts (python)
- exploring MIBs (explain some managed objects)
- setup security usign community strings
- writing your own agents