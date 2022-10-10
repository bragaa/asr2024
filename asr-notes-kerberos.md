# Overview
  - allows mutual authentication of a client and server processes and deriving a session key
  - three entities: client, server, the authentication server (hence the Kerberos mythology)
  - principals naming scheme: name[/Instance]@REALM
  - initial trust model:
    - mutual trust between AS and the principals in its realm using pre-shared secrets (the keys table)
  - cross-realms authentication
    - by establishing mutual trust between %
  - how the functional architecture of Kerberos maps to the ADDS architecture
<!--  
# Architecture

- Principals naming scheme
- tickets

# Initial trust model

# Protocol operation
- basic operation
- ticket validation
- preauthentication
- cross realms authentication

# Domain Logon Scenarii
- SSO
- Interactive domain logon
- Access to services
- Access to resources in other domains


- A Kerberos primer

slide: How Kerberos works? initial authentication and re-authentication
  - the client authenticates against the AS; it receives the TGT with additional information
  - (before service access, ) the client requests ST from the TGS
  - the client requests service access
  - The authentication occurs only once --> SSO
- Windows implementation Kerberos
  - a special program, the winlogon acquires, caches and renews the TGT on behalf of the actual principal.
  - client programs (file explorer, printing client) acquires service tickets every time they are required.
 -->

# Kerberos Physical Structure
  - Keys:
    - User keys, obtained through DES-CBC-MD5 from the user password
    - System key (not the password, but derived from a password generated automatically at domain join)
    - Service keys, derived from the krbtgt account password, shared among all the KDC
    - Session keys, derived temporarily to protect pairwise communication
  - Account passwords: user, computer, or service
  - Principals: stored in Service-Principal-Name attribute
    - User Principal Name (UPN) associated with a User account
    - Service Principal Name (SPN), associated with either a Computer or User
    - Computers automatically have SPNs
  - Accounts (from https://docs.microsoft.com/en-us/windows/win32/services/localsystem-account?redirectedfrom=MSDN):
    - LocalSystem Account
    - LocalService,
    - NetworkService,
  - Tickets: used to transport infromation to target service (via the )
  
# Architecture:
    - Microsoft implements the KDC as a single process that provides two services: AS and TGS
    - the Key Distribution Center (KDC) is implemented as a domain service.
    - the KDC uses the domain's Active Directory as its account database and gets some information about users from the global catalog.
    - The Kerberos replication is replaced by AD replication.
    - The security principal name used by the KDC for an Active Directory domain is krbtgt. An account for this security principal is created automatically when a new domain is created.
    - A password is automatically assigned to the krbtgt account. The password for the krbtgt account is used to derive a secret key for encrypting and decrypting the TGTs that the KDC issues.
- Operation:
  - Message codification: KRB_AS_REQ
  - Client -- AS
    - KRB_AS_REQ: UPN, Pre-authentication (based on the CS secretkey),
    - KRB_AS_REP: C-TGS sessionkey, TGT encrypted with the krbtgt key containing: C-TGS sessionkey, Authorization data(user SID and groups, universal groups). From now on, the user keys is replaced with the Client-TGS Session key. The TGT must be renewed as well before expiry.????
    Renewable ticket and cumulative time.
  - Client -- TGS
    - KRB_TGS_REQ: target service, the user's TGT, authenticator
    - KRB_TGS_REP: C-S sessionkey, the ST encrypted with the S-KDC secret key, containing (the C-S sessionkey, Authorization data copied from the user's TGT.)
    - Finally, (we are in the context of authorizing a domain logon), the LSA decrypts the ticket, extracts the PAC and creates (along with local information on the UPN) the access token and creates the shell
  - Client to Server
    - KRB_AP_REQ: flag indicating whether to use session key, flag indicating whether the client wants mutual authentication, ST encrypted with ..,an authenticator encrypted with the session key for the service.
    - KRB_AP_REP:
  - The authentication process for a domain user to access their computer is very similar to the process used to authenticate access to network resources—that is, the user must obtain a valid ticket for the local workstation.
  - Pre-authentication: mandatory in Windows (but can be disabled if required), it is based on assuming that fresh requests contains fresh timestamp. client sends a timestamps encrypted with its CS secretkey; the KDC if the check passes will accept the client for TGS request
  - Privilege Attribute Certificate: The Microsoft implementation follows the Kerberos specification by using the authorization field (from kerberos protocol) to store information about user identity and group membership. It is generated from the initial AS or TGS exchange (why not just AS). Comprises:
    - RIDs of the client and all the groups it belongs to
  - Authenticator is used for the access requestion validation (the ticket is just a container for the ticket). The authenticator is protected by the session key, anti-replay assured.
    - comprises: Checksum, subkey, Sequence Number, Authorization Data (not the PAC)
  - Whenever the client needs to access a service, it justs contacts the TGS with its TGT, and creates an authenticator using the Client-TGS SK.
  - inter realm authentification
    - trust: unidirectionnal, automaticall created among parent-child
    - transitive unless shortcut
    - referral tickets: one intended to a TGS in a different trusted realm, encrypted with the inter-realm key that the KDC in West shares with the other KDC.
- Content of a ticket: two parts:
  - clear part: version, issuing realm, server name,
  - protected with the tartget's key: Flags, SK, client principal name, validity information, IP addresses of client, time of initial authentication with AS (copied from TGT to ST by the TGS), authorization data (the list of SIDs of the client and its groups) aka privilege attribute certificate (PAC)
  - the credential cache holds from sign on to sign off all the tickets and the hashed secret key and session keys.
  - the access token is created from the PAC of the ticket as well as the local SAM

<!-- 
  course organization:
  start by the interaction with TGS to provide access to the server.
  show access to the local machine (as Host service principal)
 -->