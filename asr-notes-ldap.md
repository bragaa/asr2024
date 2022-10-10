# What are directories?
- (en français: annuaire)
- simply viewed, a **directory** is a database of objects, organized hierarchically. Each object is a collection of valued attributes and represents an information from any business field (en employee, a department, a person, a computer, etc.)
- a **directory service** is a network-wide service, used to publish the directory and make it available on a network to clients through the use a specific access protocol, the widely used being the **Lightweight directory access protocol** (LDAP), a variation of X.500 DAP.

# Where directories are being used?
- The YellowPages is an example of directories: collection of professionnals, seachable with a key (work, location, name, etc.)
- In computing, storing employee related information like phone, emails, configuration parameters for your phone, email and other clients which must be shared among multiple distributed applications is typically done through an ldap network service.
- directories are just another way to store data, just like RDBMS, however for historical reasons, it has taken a distinct path and we cannot get rid of it!
- Historically,
  1. directories were the only one suitable for hiearchical data (not RDBMS).
  2. access to data was done through a standard protocol (as opposed to database-specific tools)
  3. data is more read that written, the directories storage engine was optimized for read speed more than RDBMS
- Common applications of LDAP directories are:
  - as a searchable datastore, available throughout a network
  - identity authentication (discussed later)
- Major software companies provide LDAP-compatible directory servers as part of their product suite :
  - Microsoft: Active Directory, Azure Active Directory; IBM: Tivoli DS; RedHat: RH DS and FreeIPA
  - In those cases, LDAP directories are used to store and publish configuration information related to various corporate network elements (users, computers, services, etc.), and also used in identity authentication (use for example in case of worktation logon).
  - Example: Microsoft use Active Directory (AD) to store attributes of various corporate network objects (user, workstations, network services like DNS, mail servers, etc.).
    - at workstation startup or user authentication, the system looks for the active directory server and retrieves various customizations for the desktop, the network shares, network information, etc. AD is also used to store accounts' passwords.
- Open source implementations of LDAP exists as well: OpenLDAP, ApacheDS, 389 (by RedHat)
- When developing your own application, you can use an LDAP directory server to store hierarchically structured data. You can also use it to store identities if an authentication is to be performed. In this case, the directory server is used in a so-called application mode, accessible only to your application: Microsoft AD Application Mode (ADAM), Python's ldaptor library)
- Third party security devices (e.g. Fortinet VPN end points) can (according to config) authenticate users request specific network services (VPN establishment) against Active Directory accounts
  - match users, perform bind with adequate mechanism and check the result
- Cisco IP Phones can leverage existing LDAP-compatible servers for name-to-phone-number mapping
- BIND and ISC DHCP offer the possiblity to use LDAP to store their data, as an alternative to using text files.
- Example: Apache may also provide authentication against LDAP service

<!--![Excerpt of ADDS content from Windows Server 2003](../imgs/ldap-detailedW2k3ServerLDAPDIB.dia)-->

# LDAP Directory architecture
- uses a client-server model. clients (resp. servers) are named Directory user (resp. server) agents (DUA, resp. DSA). server-to-server communication is also defined to perform replication, resolve referrals, etc.

- LDAP standard defines the (1) communication protocol: the operations a directory server must support, the messages encoding, the protocol security, etc. (2) the data representation and naming, (3) standard data types (integers, etc.) and the operations (intersects, union, etc.) (3) the replication, among others.

- aspects which does not impact interoperation between LDAP clients and servers are left to the implementation, for example, (1) the actual data storage engine; is it relational database, hdb database, LDIF files, etc, (2) the access control to the directory objects, etc.

- a directory is a collection of objects (aka entries) organized hierarchically. clients uses LDAP in order to access the data and perform any of the allowed operations (search, modify, etc.). The root of the hierarchy is called root DSA specific entry (rootDSE)

- for operational reasons (load balancing, security, reduce access time), the directory objects may be distributed among multiple directory servers (as in active directory); a chaining or referral mechanisms allows the clients to access requested objects independantly from the contacted DSA.

<!--![LDAP Architecture](../imgs/ldap-architecture.dia)-->

# LDAP naming model
- each directory object:
  - is a set of (attribute_name: attribute_value) pairs
  - is an instance of one or more directory object classes which define its attributes (can be found in the object attribute objectClass).
  - is identified within its parent by its relative distinguished name (rdn), and identified throughout the whole directory by the by the dn. the dn=rdn + dn of the parent object.
  - (the root object called **rootDSE**). Each object is uniquely identified by a name; **the distinguished name** (dn) and by a relative name within its siblings (rdn), the dn is the contactenation of the rdn and the dn of the parent object. Each object has its structure defined by a class, and the classes used in a directory make up the directory subschema.

- Each directory has its structure defined by the **directory schema** (generally referred to as subschema) which defines (among others):
  - the objects classes used in the directory (their object identifier (OID), the mandatory and optional attributes, whether attributes can hold multiple values, type of the class: auxiliary/structural/top, the parent class it is derived from if any)
  - the attributes types, the attributes syntaxes and the matching rules thereof. (matching objects depends on the attribute syntax (integers, strings, etc.))
  - the name forms (which attributes are allowed as rdn for each structural class)
  - the content rules (defines the auxiliary classes allowed with each structural class). auxiliary classes are used to decorate classes with additional attributes (can think of it as secondary object class, the rdn attributes are not part of the secondary classes).
  - the structure rules, defines the allowed superior structural classes for an entry.
- Each schema element (classes, name forms, content rules, etc.) is universally identified with an Object ID (although it is not an object). They are defined by organizations within their own namespaces.
- an object belongs to exactly one structural class, and possibly many auxiliary classes.

- Example of an openldap directory object (of class inetOrgPerson).
```
  dn: cn=Robert Smith,ou=people,dc=example,dc=com
  objectclass: inetOrgPerson
  cn: Robert Smith
  cn: Robert
  sn: Smith
  uid: rsmith
  mail: robert@example.com
  mail: r.smith@example.com
  ou: sales
```

- LDAP defines two special classes: alias and top
- in addition to user data, objects contains operational (aka administrative) attributes (e.g. objectClass, createTime, lastAccessTime, attributes related to access control).These are not actual user data and not returned by default to the clients.
- It is possible to define new object classes

# LDAP operations
- LDAP service runs on port tcp/389 by default. The SSL-enabled version (ldaps) uses tcp/636.
- Standard protocol operations include: search, compare, add, modify, delete, and rename for objects management.
- a session is typically required before accessing objects. Standard protocol operations for session management are bind, unbind, and startTLS.

- an LDAP server may allow anonymous access, clients just needs to send bind operation with no authentication credentials, or even access the object directly (Microsoft AD uses an anonymous version of LDAP named Connection less LDAP (CLDAP) which does not require bind operation).

- A typical session workflow is as follows:
  - a client issues a bind; the session can be anonymous or authenticated.
  - then performs and allowed operation: search, compare, add, etc.
  - then closes session with unbind.
- Client can choose to establish a TLS tunnel, to tunnel all the session. authentication for the tunnel is based on certificate.

- LDAP protocol is extensible in various ways:
  - the default behaviour of operations can be modified using operation controls (available in the rootDSE object attributes)
  - new operations can be supported using as Extended operations.
- as discussed previsously, an operation typicall only takes place within a session: the client must bind, issue the desired operation, then optionally unbind.
- search operation is performed against a whole subtree of the diretory hierarchy. search arguments which must be specified by a client are:
  - baseDN: the dn of the subtree root object
  - scope, depth relative to the baseDN, one of "base" | "one" | "sub"
  - derefAliases: should the DSA follows aliases (alias objects are just like a symbolic link to abjects in an other DSA)
  - size and time limits for response
  - attrsOnly
  - filters, using operators ~=, =,  \*, |, &
  - list of attributes
- Example of filters:
  - (&(objectClass=person)(|givenName=John)(mail=john*)))
  - (&(objectClass=person)(|givenName=John)(mail:caseExactMatch:=john*)))
- Examples with the `ldapsearch` (ldap client command line implementing search operation):
```
    ldapsearch -h myServer -p 5201 -D cn=admin,cn=Administrators,cn=config -W
      "(cn=Charlene Daniels)"

    ldapsearch -h bluepages.ibm.com -p 391 -b "cn=HR Group,ou=Asia,o=IBM" -s base -l 300
        "(objectclass=*)" member

    ldapsearch -h bluepages.ibm.com -p 391 -b "o=ibm" -l 300 -z 1000
        "(&(objectclass=Person)(|(cn=mary smith*)(givenname=mary smith*)(sn=mary smith*)
          (mail=mary smith*)))" cn
```

# Replication and Partitioning
- the topmost object in a directory is a special object named the rootDSE (having attributes describing the directory: supported security features, ldap version, directory owner, etc.). Just below the rootDSE comes one or more directory partitions; a directory partition is an objects hierarchy, generally starting with an object of class `domainComponent` (as for the directory data partition in AD). The partition defines the _naming context_ for all the directory objects that comes below it, that is to say, the dn of all descendant objects have that dn in their own dn. This objects hierarchy is called: naming context (nc), partition, or directory information tree (DIT).
- a DSA can host multiple naming contexts, all attached to the rootDSE. Those naming contexts can have a common suffix in their dn or not.
- the goal of partitionning the directory is to make it possible to host different partitions of the same directory to be host in distinct DSAs. This feature makes it possible to meet security or performance requirements. use cases:
  - deploy partition where it is consumed the most
  - make a sensitive part of the directory be hosted on certain more secure locations.
- LDAP defines replication between multiple DSAs in order to meet response time or operational requirements. Replication is defined on a per-partition base: each partition can be replicated among a specific set of DSAs and with a specific timing/bandwidth configuration.
- Example with AD:
  - all the objects in a corporate network (computers, users, groups, policy objects, etc.) are stored within the corporate directory. Multiple DSAs are deployed to support large networks.
  - For administrative purposes, those objects can be grouped into disjoint Windows Domains and replicated among those DSAs responsible for the domain. The domain objects are thus replicated within the scope of those DSAs.
  - However, Active Directory requires that a partition called the Global Catalog (gc), containing a 'summary' of the entire directory objects be shared among all the domains DSAs (the entire forest). The same thing applies for the schema partition (shared among the forest domain controllers).

# LDAP authentication
- LDAP defines different techniques for the DSAs to authenticate clients:
  - *Anonymous Bind*: no credentials are given in the Bind message. (or also no bind message at all)
  - *Simple Bind*: client provides a dn representing the subject identity and a secret for authentication. The secret is sent in clear text; to protect it, the client may establish an encrypted TLS tunnel using the `startTLS`
  - *SASL Bind*: authentication is deleguated to the SASL entity. SASL is an extensible layer within LDAP protocol providing various security mechanisms to LDAP: Digest-MD5, GSSAPI/Kerberos, NTLM, etc.
- startTLS operation is used in order to prepare a tunnel before performing Bind operations using credentials out of the scope of LDAP.
- Clients may send operations requests without performing a bind if anonymous bind is supported.
- Example: Check which SASL mechanisms are available on the AD server. Perform query on the rootDSE or use the relevant LDAP client library function in python.
- Mutual authentication is only possible with some SASL mechanisms or with the startTLS operation

# LDAP Data Interchange Format (LDIF)
- LDAP Data Interchange Format (LDIF) is a standard text files format for representing directory objects and object manipulation operations.
- use cases:
  - for batch processing (add, modify, etc.): a sysadmin prepare objects definition in LDIF using different tools and import them within the DSA.
  - import/export between DSAs from different providers

```bash
dn: CN=Foulen Elfouleni,OU=Legal,DC=example,DC=com
changetype: modify
replace:employeeID
employeeID: 1234
-
replace:employeeNumber
employeeNumber: 98722
-
```

# LDAP URLs
- **LDAP URLs** (defined in RFC2255) are strings that encode a search operation against a DSA. They allow LDAP-enabled browsers to perform an ldap search.
- The general formula for an LDAP URL is: `ldap[s]://<hostname>:<port>/<base_dn>?<attributes>?<scope>?<filter>`, where:
  - base_dn: dn for the subtree topmost object where the search takes place.
  - attributes used to tell the desired attributes
  - scope defines the search depth: `base`, `one`, `sub`
  - filter express the search criteria
- Example: ```ldap://notre-serveur-ldap.rnu.tn:389/dc=enig,dc=rnu,dc=tn?sn,mobilePhone?sub?(sn=F*)``` shows the sn and mobilePhone attributes for all entries in the DIT at the server root having sn starting with 'F'.
- LDAP URL is suited to encode attributes resulting from parametrized search operation.

# Misc
- **Alias** objects can be used to add a different dn to an object. (class Alias)
- **Referrals** are used in directory to link between different directory partitions.
- starting with LDAPv3, each DSA must publish its schema online allowing clients to dynamically decode the directory objects. the used schema are published in the attribute subschemaSubsentry of the rootDSE.
- several LDAP implementations defines object access control based on **access control lists** (ACLs). This is not a standard feature, so no standard schema to represent permissions, rules, etC.
  - Example (OpenLDAP):
    `access to attrs=userPassword
    by self =xw
    by anonymous auth
    by * none`
    `access to *
    by anonymous none
    by * read`
    - The general format of an ACL is `<target> <permission> <bind rule>`, where `target`: a directory subtree, permission: (r, w, etc.) and bindrule is the bindDN used to identify the subject.
- openLDAP supports directory-based configuration, where some of the server parameters and the subschema are described in a dedicated naming context.
- Example:
  ```
  dn: cn=schema,cn=config
  objectClass: olcSchemaConfig
  cn: schema

  dn: cn=test,cn=schema,cn=config
  objectClass: olcSchemaConfig
  cn: test
  olcAttributeTypes: ( 1.1.1
    NAME 'testAttr'
    EQUALITY integerMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.27 )
  olcAttributeTypes: ( 1.1.2 NAME 'testTwo' EQUALITY caseIgnoreMatch
    SUBSTR caseIgnoreSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.44 )
  olcObjectClasses: ( 1.1.3 NAME 'testObject'
    MAY ( testAttr $ testTwo ) AUXILIARY )
    ```

# Assignments: Explore the Microsoft AD directory
Tout au long de ce travail, vous allez explorer l'annuaire AD de Microsoft.

1. Depuis un contrôleur de domaine, lancez le programme "adsiedit", qui n'est autre qu'un client ldap. connectez vous au serveur ldap local avec votre compte d'admin du domaine.
2. Vous obtenez une arboresence, dont la racine est rootDSE (non indiquée). Déterminez les informations suivantes:
  1. les "naming contexts" (partitions d'annuaire) présents sur cet annuaire ?
  2. les techniques d'authentification supportées (attribut du rootDSE)
  4. quelle est la classe des deux objests suivant : "cn=gcr,cn=users,dc=enig,dc=tn"
  5. le nombre d'ordinateurs et d'unités d'organisation renseignés dans cet annuaire ?
3. (supprimée) Placez une machine linux dans le même réseau que le contrôleur de domaine. Retrouvez avec la commande `ldapsearch` depuis la machine linux les réponses aux questions 2.1 et 2.5.
4. Retrouvez les utilisateurs par défaut dans un domain windows : objets de la classe User dans le noeud Builtin "cn=builtin,dc=enig,dc=tn").
4.5.  Localisez la parition contenant le schéma utilisée dans votre serveur. (si nécessaire, rechercher sur le sie technet.microsoft.com)
5. Retrouvez la description textuelle de la classe person, dans l'exploraeur de schéma.
7. Reconnectez vous en tant que gcr, essayez de créer un objet dans le container Users. Quel message apparait ? Essayez de modifier un attribut quelconque (non opérationnel).
6. Refaite la même action avec un compte "cn=Administrateur,dc=enig,dc=tn".
8. Vous allez maintenant préparer un fichier LDIF pour ajouter un lot d'utilisateurs. Pour cela,
  0. Exportez en utilisant la commande ldifde (sur windows server) l'objet utilisateur "cn=gcr,dc=enig,dc=tn" en format ldif. prenez-le comme base et modifiez les attributs nécessaires.
  1. Retrouvez le format de ldif pour coder une opération d'ajout : quel éléments doivent y apparaitre?
  2. Utilisez ce format pour ajouter au fichier au moins un autre user.
  4. Réalisez l'opération d'ajout (avec la commande ldifde)
9. Faut il renseigner les attributs opérationnels lors de l'ajout d'un objet ? vérifiez-le.
11. (supprimée) Essayez de vous connecter à l'objet rootDSE avec ldapsearch sans donner des mots de passe
12. (supprimée) Notez les codes d'erreurs lorsque vous vous connectez avec les situations suivantes :
  6. vous spécifier un mot de passe invalide
  7. vous spécifier une méthode d'authentification non supportée
  8. vous effectuez une opération non autorisée comme la suppression d'un objet ou de son attribut
15. Localisez les ACLs. (recherchez sur Internet)

# Assignment
- Use LDAP authentication in your applications (use AD Connect to use cloud AD to authenticate the on-prem clients.

# Reading materials
- http://www.zytrax.com/books/ldap/
- http://www-sop.inria.fr/members/Laurent.Mirtain/ldap-livre.html (fr)
- http://ldapwiki.com
- http://ldapman.org


<!--
------
# Syllabus
- directories (as a database) and the directory access protocol (LDAP), how different from relational db
- ldap architecture (dua, dsa, used ports, storage backend, integration with other DSAs)
- data model: objects, directory information base (DIB) layout (rootDSE, multiple DITs/naming contexts, subschema, and the hierarchical organization), naming scheme (dn/rdn, how the attribute serving as a rdn is selected)
- elements of the directory schema: structure rules, syntaxes, attributes,classes, type of classes, operational vs user attributes, referrals objects
- prototcol commands: search, bind, unbind, startTLS, add, delete, modify, rename, etc.
- how search works within a directory informationt tree (DIT)
- directory data replication and partitioning: naming context, referrals, replication protocol
- authentication methods: anonymous bind, simple bind, and SASL bind, startTLS
- miscellaneous items: LDIF (usage, format), LDAP URLs, access control (although this is not a standard feature)
- be able to use a directory-like storage from your application, select a subschema from standard classes
- be able to use bind request for authentication purpose, perform searches



Background on the LDAP replication technologies
-
- if we have to maintain multiple DSAs, so they must have synchronized DIT fragments. So why we need to have multiple? for multiple sites (link usage optimization), load distribution, resiliency
- common data replication approaches involves master-slave or the multi-master replication
- If multiple DSA were allowed to be written, then some form of concurrency control or conflict resolution must be afforded for; Some forms of conflicts that may happen are:
-- the same attribute is being changed on many masters
-- one parent is being removed whithout any explicit decision on the child objects




LDAP replication technologies
- IETF: Sync Operation (https://tools.ietf.org/html/rfc4533)
- proprietary: use of RPC over IP, SMTP, LDAP (for initial replication in ADDS)

- Sync Operation : (RFC4533)

# Excercices:
-->