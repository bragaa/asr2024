# study case
- You were hired as a sysadmin in an entreprise (an SME) because the IT infrastructure is getting larger. Previously managed by a general technician with no specific IT knowledge (doing his best effort).
- There are several issues in the network: multiple network shares and printers across the users workstations, multiple security incidents (viruses are spreading from time to time, fearing the worst), user accounts management is consuming time (passwords lost, must be renewed,), some important files were lost, manager is loosing control of some sensitive files permissions because permissinos must be configured on each share.

- The network is made up of 2 servers (a GMAO solution and Zimbra as mail server), 10+ desktop computers, printers, switche, and an ISP CPE. multiple user accounts.

- you are asked to suggest improvements

# key points
- IT infrastructure supports business. For an entreprise, the IT infrastructure and associated costs (CAPEX and OPEX) must be optimized to meet the business requirements. No less, No more.

- **risks** facing the **entreprise's assets** must be evaluated in order to install the adequate **security controls**. assets are either services (functionnalities) or data. data assets are prioritized according to the sensitivity (cost if leaked or lost). service assets are prioritized according to availability (cost if unavailable).

- key ideas to help improve the situation (some will be explained in further classes): active directory, firewall (fw), proxy, segmentation,

- when requesting acces to a resource from a server, a user (through a user agent, aka client software) goes typically through **authn** step then an **authz** step. a **session key** shared by client and server is computed at the end of the authn. used to protect subsequent messages exchanged during the session by using a **HMAC** field appended to the message. HMAC is somehow similar to a CRC/MIC, but involves the session key to help prove the message authenticity to the received. authencity = message integrity and source authentication. Common  algos used in HMAC are MD5 and SHA1. It depends on the actual protocol used to access the ressource.

- authn involves credentials of different types; passwords/biometrics/one time passwords/certificates (using public key crypto), and is done using different methods: clear passwords (e.g. PAP being used in PPP, Basic being used in HTTP), challenge-based (e.g. CHAP used in PPP, NTLM used in windows environments), public key cryptos, etc.

- authz consists in determining the permissions of a given user after being authenticated. permissions can be configured in the server by following models: associated permission fo each user, associated permission for each role (Role based access control) and the role of the user, associate permissinos using a list of rules (Rule based access control).

- with regard to security, a fw can be used. role of the fw is just a packet filter. but in industry the term firewall goes beyond mere pf and refers to a device implementing multiple security features.
	- pf are based on ACLs. filters packets based on l3 and l4 criteria. ACLs generally implement secure by default (default rule leading a deny if not explicitly modified.)
	- pf cannot inspect beyon l4.

- in order to inspect (and control) internet access by employees, a **proxy** can be used. web proxy can be explicit (splits the TCP connection between c and s, and relay TCP payload after inspecting and applying rules) or transparent. proxy can also be used on the server side, and is known as **reverse proxy**. (the former is also called forward proxy to prevent confusion).

- a network share term refers to a file folder shared across the network. users accessing a network share needs an account on it.

- accountable for vs responsible for.

- common entreprise applications supporting their business: Entreprise resrouces planning (ERP), Enterprise assets management/computarized maintenance management systems (CMMS), Workflow management systems (WFMS), Customers relationship management system (CRM), Enterprise content management (ECM), etc.
- **business process** or **business workflow** expresses the flow of information (work orders, feedbacks, reports, sms/mails notifications,) among multiple persons collaborating on a business within an entreprise. A paperless flow. Collaboration made based on a software implementing the workflow.
	- BP can be modelled using graphical languages like BPMN (business process modelling notation). WFMS are software we use in order to automatically create an application from a business process definition (in BPMN). examples: Bonita,

