# OpenLDAP Cookbook
Tested under Ubuntu 22.04

## Abbreviations
- "slapd" = Stand-alone LDAP Daemon
- "CN" = Common Name (name of an LDAP object)
- "DN" = Distinguished Name (definite path to an LDAP object)
- "OU" = Organizational Unit (container for LDAP objects, e.g. a department of your company)
- "LDIF" = LDAP Data Interchange Format
- Class = "template" for an LDAP object

## Installation
```
# install slapd with contrib overlays and ldapvi for easy command line administration
# setup recognizes domain automatically based on /etc/hosts and creates a default database for it (dc=example,dc=com)
# setup will ask for an admin password
apt install slapd slapd-contrib ldap-utils ldapvi
```

Two admins are created automatically:
- schema/config admin: `cn=admin,cn=config`
- domain admin:        `cn=admin,dc=example,dc=com`

Don't forget to replace `dc=example,dc=com` in the examples with your domain!

## Connect
### Via Local Unix Socket
You can use `ldapvi` to explore the LDAP tree and make instant changes on the command line, authenticated passwordless as root (via `-Y EXTERNAL`) using the local Unix socket `-h ldapi:///`.
```
# use "-b cn=config" to view the config tree
ldapvi -Y EXTERNAL -h ldapi:/// -b cn=config

# use "--discover" to view the first domain found
dapvi -Y EXTERNAL -h ldapi:/// --discover
```

### Via LDAP(S) - External Tools
If you connect from external hosts with tools like Apache Directory Studio, you need to enter the full user DN as username ("user@domain" as known from Active Directory is NOT working)!

## Rename Admin
To exacerbate brute force attacks, you should rename the admin accounts.

Note: most OpenLDAP configuration resides directly within the LDAP tree below `cn=config`, this is called "dynamic config" and is extremely useful since it can be easily replicated and backed up with the same techniques used for regular LDAP entries.

<details>
  <summary>Dynamic Config</summary>

```
$ ldapmodify -Y EXTERNAL -H ldapi:///
dn: olcDatabase={0}config,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=admina,cn=config

# press return 2 times
```
</details>

## Enable TLS (LDAPS)
If you want to enable LDAPS, you need a valid certificate. Set the path to the cert and key file in the dynamic config.
<details>
  <summary>Dynamic Config</summary>

```
$ ldapmodify -Y EXTERNAL -H ldapi:///
dn: cn=config
changetype: modify
add: olcTLSCertificateFile
olcTLSCertificateFile: /etc/ldap/certs/cert.pem
-
add: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/ldap/certs/key.pem
-
add: olcTLSCipherSuite
olcTLSCipherSuite: SECURE256:+SECURE128:-VERS-TLS1.0:-VERS-TLS1.1
```
</details>

<details>
  <summary>Enable TLS/LDAPS:// and disable LDAP://</summary>

At this point, clients can use STARTTLS through port 389. To enable TLS/LDAPS on Port 636, you need to edit the config file `/etc/default/slapd`:
```
SLAPD_SERVICES="ldap:/// ldaps:/// ldapi:///"
```
You can remove `ldap:///` to disable unencrypted communication completely.

Restart `slapd` after changing the file.
</details>

## Disallow Anonymous LDAP Bind
By default, everybody can read the complete LDAP tree via anonymous bind. You should disable this.
<details>
  <summary>Dynamic Config</summary>

```
$ ldapmodify -Y EXTERNAL -H ldapi:///
dn: cn=config
changetype: modify
add: olcDisallows
olcDisallows: bind_anon
```
</details>

## Permissions / Access Control Lists
An ACL looks like: `to <WHAT> by <WHO> <PERMISSION>`. The `by <WHO> <PERMISSION>` can be repeated multiple times.

| \<WHAT\>        | Description                        | Aliases           |
| --------------- | ---------------------------------- | ----------------- |
| dn=...          | the exact object                   | dn.exact, dn.base |
| dn.one=...      | this object and one level below    | dn.onelevel       |
| dn.sub=...      | this object and all sub objects    | dn.subtree        |
| dn.children=... | all sub object but not this object |                   |
| *               | all objects                        |                   |
| attrs=...       | specific attributes                |                   |

| \<WHO\>         | Description                                      |
| --------------- | ------------------------------------------------ |
| anonymous       | not logged in user (anonymous bind)              |
| users           | all logged in users                              |
| self            | self-permission (e.g. for changing own password) |
| group           | permission for group members                     |
| *               | all objects                                      |

| \<PERMISSION\>  | Description                                      |
| --------------- | ------------------------------------------------ |
| none            | no access                                        |
| auth            | access while bind (necessary for authentication  |
| read            | read-only                                        |
| write           | change, add or delete entries                    |
| manage          | full access                                      |
| compare         |                                                  |
| search          |                                                  |

The order of ACLs are important. The first matching rule is used to determine access to an object or attribute. Following rules will not be processed except if the matching rule has the "break" statement.

<details>
  <summary>Example: add permission to your domain tree for the root user</summary>

In this example, we give our root user permission to manage the domain database too and not only the config tree. The rule is inserted as first entry {0} and since we only want to target the root user via EXTERNAL auth, we append a `by * break` so that for every other user the following rules were applied.
```
$ ldapmodify -Y EXTERNAL -H ldapi:///
dn: olcDatabase={1}mdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to dn.sub=dc=example,dc=com
  by dn.exact=gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth manage
  by * break
```
</details>

<details>
  <summary>Pre-defined ACLs explained</summary>

```
{1} to attrs=userPassword
  by anonymous auth  # necessary to make authentication (bind) work
  by self write      # allow users to change the own password
  by * none          # disallow to view password hashes of other users

# same applies for the password changed date attribute
{2} to attrs=shadowLastChange
  by anonymous auth
  by self write
  by * none
```

I recommend changing the last default rule `to * by * read` to `to * by users read` to disallow read access for anonymous binds.
</details>

<details>
  <summary>Permissions for group members</summary>

This example let the member of an admin group add/delete/change the objects below `ou=external-users,dc=example,dc=com`:
```
{3} to dn.children=ou=external-users,dc=example,dc=com
  by group/groupOfNames/member=cn=ADMIN-external-users,dc=example,dc=com write
  by * break
```

If you want to enable your admin group members to reset passwords, you need to modify the default ACL to:
```
{1} to attrs=userPassword
 by self write
 by anonymous auth
 by group/groupOfNames/member=cn=ADMIN-external-users,dc=example,dc=com write
 by * none
```
</details>

## Overlays
By default, the OpenLDAP core is only a "data storage". Every other logic needs to be added through extensions, called "overlays".

### Dynlist (MemberOf)
You surely want to have a `memberOf` attribute displayed on user account so you can easily query in which groups a user is member of. Earlier, there was a separate "memberof" overlay, but this is deprecated. You now use "dynlist" with appropriate configuration.

<details>
  <summary>Setup</summary>

First, import the schema necessary for dynlist: `sudo -u openldap slapadd -b cn=config -l /etc/ldap/schema/dyngroup.ldif`.

Then, load the overlay:
```
$ ldapmodify -Y EXTERNAL -H ldapi:///
dn: cn=module{0},cn=config
changetype: modify
add: olcModuleLoad
olcModuleLoad: dynlist.la
```
You may need to restart `slapd` at this stage.

Now create a config telling dynlist to dynamically render a `memberOf` attribute by the `member` attribute of the `groupOfNames` object class:
```
$ ldapmodify -Y EXTERNAL -H ldapi:///
dn: olcOverlay={1}dynlist,olcDatabase={1}mdb,cn=config
changetype: add
objectClass: olcOverlayConfig
objectClass: olcDynListConfig
olcOverlay: {1}dynlist
olcDynListAttrSet: {0}groupOfURLs memberURL member+memberOf@groupOfNames
```
</details>

<details>
  <summary>Test</summary>

You can test this by creating a group (groupOfNames), a user (inetOrgPerson) and add the user to the group (by adding the user DN to the "member" attribute of the group):
```
$ ldapmodify -Y EXTERNAL -H ldapi:///
dn: cn=tester,dc=example,dc=com
changetype: add
cn: tester
sn: Testperson
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: top
 
dn: cn=mygroup,dc=example,dc=com
changetype: add
member: cn=tester,dc=example,dc=com
cn: mygroup
objectClass: groupOfNames
objectClass: top
```

Now, when querying the user account, you automatically get a "memberOf" attribute with the DNs of all groups where the user is member of. Note that "memberOf" is an operational attribute. In Apache Directory studio, it must be configured to display them.
</details>

### Refint
Referential Integrity automatically updates the user DN in the "member" attribute of all corresponding groups when the CN of a user is changed.

<details>
  <summary>Setup</summary>

Load the module:
```
dn: cn=module{0},cn=config
changetype: modify
add: olcModuleload
olcModuleload: refint.la
```

Configure it:
```
dn: olcOverlay=refint,olcDatabase={1}mdb,cn=config
changetype: add
objectClass: olcOverlayConfig
objectClass: olcRefintConfig
olcOverlay: refint
olcRefintAttribute: memberURL uniqueMember member
olcRefintNothing: cn=dummy,dc=example,dc=com
```

Please make sure that the object given as "olcRefintNothing" exists. As per definition, an LDAP group must have at least one member. If you delete all user accounts of a group, Refint will set this dummy user as new "member".
</details>

### PPolicy
Password Policy enforcement. Also enables you to set an expiration date for a user account.

<details>
  <summary>Setup</summary>

Load the module:
```
dn: cn=module{0},cn=config
changetype: modify
add: olcModuleLoad
olcModuleLoad: ppolicy.la
```

Configure it and create a default policy:
```
dn: olcOverlay=ppolicy,olcDatabase={1}mdb,cn=config
changetype: add
objectClass: olcOverlayConfig
objectClass: olcPPolicyConfig
olcOverlay: ppolicy
olcPPolicyDefault: cn=default,ou=Policies,dc=example,dc=com
olcPPolicyForwardUpdates: TRUE
olcPPolicyHashCleartext: TRUE
olcPPolicyUseLockout: TRUE

# create an OU for policies
dn: ou=Policies,dc=example,dc=com
changetype: add
objectClass: organizationalUnit
ou: Policies

# create the default policy
dn: cn=default,ou=Policies,dc=example,dc=com
changetype: add
objectClass: pwdPolicy
objectClass: person
cn: default
pwdAttribute: userPassword
pwdAllowUserChange: TRUE
pwdExpireWarning: 1440
pwdGraceAuthnLimit: 5
pwdInHistory: 5
pwdLockout: TRUE
pwdFailureCountInterval: 300
pwdLockoutDuration: 3600
pwdMaxFailure: 5
pwdMinLength: 12
pwdCheckQuality: 1
sn: OurDefaultPolicy
```
</details>

<details>
  <summary>Set user account expiration date</summary>

  You can set the operational attribute "pwdEndTime" of a inetOrgPerson account to disable the login automatically at the given time without deleting it. The user account can then easily be reactivated.
</details>

## Backup and Restore
<details>
  <summary>Backup Script</summary>

`slapcat` works like `mysqldump`. It creates a plaintext dump of your LDAP database which can be restored on any OpenLDAP version. You can put this in your crontab.

```
#!/bin/bash
set -e
BACKUP_PATH=/backup
slapcat -b cn=config > ${BACKUP_PATH}/config.ldif
slapcat -b dc=example,dc=com > ${BACKUP_PATH}/example.com.ldif
```
</details>

<details>
  <summary>Restore Script</summary>

The config tree is stored in `/etc/ldap/slapd.d/` and the domain databases inside `/var/lib/ldap/*`. You need to purge both before importing a dump.

```
#!/bin/bash
set -e
 
service slapd stop
 
########## PURGE EXISTING DATA ###########
rm -rf /etc/ldap/slapd.d/* /var/lib/ldap/*
##########################################
 
slapadd -F /etc/ldap/slapd.d -b cn=config -l /backup/config.ldif
slapadd -F /etc/ldap/slapd.d -b dc=example,dc=com -l /backup/example.com.ldif
 
chown -R openldap:openldap /etc/ldap/slapd.d/
chown -R openldap:openldap /var/lib/ldap/
 
service slapd start
```
</details>

## Custom Attributes and Object Classes
Add something like the following example into your dynamic config to customize your schema.

<details>
  <summary>Dynamic Config</summary>

```
dn: cn=myownschema,cn=schema,cn=config
objectClass: olcSchemaConfig
cn: myownschema
olcAttributeTypes: {0}( 1.1.3.5.1 NAME 'myCustomAttribute' DESC 'very cool' EQUALITY caseIgnoreMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 )
olcObjectClasses: {0}( 1.1.3.7.1 NAME 'myCustomObjectClass' DESC 'even cooler' SUP inetOrgPerson STRUCTURAL MAY ( myCustomAttribute ) )
```
</details>

<details>
  <summary>Syntax OIDs</summary>

| Description                     | OpenLDAP OID                  | MS AD OID   |
| ------------------------------- | ----------------------------- | ----------- |
| UUID                            | 1.3.6.1.1.16.1                |
| "ACI" Item                      | 1.3.6.1.4.1.1466.115.121.1.1  |
| Access Point                    | 1.3.6.1.4.1.1466.115.121.1.2  |
| Attribute Type Description      | 1.3.6.1.4.1.1466.115.121.1.3  |
| Audio                           | 1.3.6.1.4.1.1466.115.121.1.4  |
| Binary                          | 1.3.6.1.4.1.1466.115.121.1.5  |
| Bit String                      | 1.3.6.1.4.1.1466.115.121.1.6  |
| Boolean                         | 1.3.6.1.4.1.1466.115.121.1.7  | 2.5.5.8
| Certificate                     | 1.3.6.1.4.1.1466.115.121.1.8  |
| Certificate List                | 1.3.6.1.4.1.1466.115.121.1.9  |
| Certificate Pair                | 1.3.6.1.4.1.1466.115.121.1.10 |
| Country String                  | 1.3.6.1.4.1.1466.115.121.1.11 |
| DN                              | 1.3.6.1.4.1.1466.115.121.1.12 | 2.5.5.1
| Data Quality Syntax             | 1.3.6.1.4.1.1466.115.121.1.13 |
| Delivery Method                 | 1.3.6.1.4.1.1466.115.121.1.14 |
| Directory String                | 1.3.6.1.4.1.1466.115.121.1.15 |
| DIT Content Rule Description    | 1.3.6.1.4.1.1466.115.121.1.16 |
| DIT Structure Rule Description  | 1.3.6.1.4.1.1466.115.121.1.17 |
| DL Submit Permission            | 1.3.6.1.4.1.1466.115.121.1.18 |
| DSA Quality Syntax              | 1.3.6.1.4.1.1466.115.121.1.19 |
| DSE Type                        | 1.3.6.1.4.1.1466.115.121.1.20 |
| Enhanced Guide                  | 1.3.6.1.4.1.1466.115.121.1.21 |
| Facsimile Telephone Number      | 1.3.6.1.4.1.1466.115.121.1.22 |
| Fax                             | 1.3.6.1.4.1.1466.115.121.1.23 |
| Generalized Time                | 1.3.6.1.4.1.1466.115.121.1.24 |
| Guide                           | 1.3.6.1.4.1.1466.115.121.1.25 |
| IA5 String                      | 1.3.6.1.4.1.1466.115.121.1.26 | 2.5.5.5
| INTEGER                         | 1.3.6.1.4.1.1466.115.121.1.27 | 2.5.5.9 (32bit)
|                                 |                               | 2.5.5.16 (64bit)
| JPEG                            | 1.3.6.1.4.1.1466.115.121.1.28 |
| Master And Shadow Access Points | 1.3.6.1.4.1.1466.115.121.1.29 |
| Matching Rule Description       | 1.3.6.1.4.1.1466.115.121.1.30 |
| Matching Rule Use Description   | 1.3.6.1.4.1.1466.115.121.1.31 |
| Mail Preference                 | 1.3.6.1.4.1.1466.115.121.1.32 |
| MHS OR Address                  | 1.3.6.1.4.1.1466.115.121.1.33 |
| Name And Optional UID           | 1.3.6.1.4.1.1466.115.121.1.34 |
| Name Form Description           | 1.3.6.1.4.1.1466.115.121.1.35 |
| Numeric String                  | 1.3.6.1.4.1.1466.115.121.1.36 | 2.5.5.6
| Object Class Description        | 1.3.6.1.4.1.1466.115.121.1.37 |
| OID                             | 1.3.6.1.4.1.1466.115.121.1.38 | 2.5.5.2
| Other Mailbox                   | 1.3.6.1.4.1.1466.115.121.1.39 |
| Octet String                    | 1.3.6.1.4.1.1466.115.121.1.40 | 2.5.5.3 (Case-sensitive general string)
|                                 |                               | 2.5.5.10 (String of bytes)
|                                 |                               | 2.5.5.12 (Unicode string)
| Postal Address                  | 1.3.6.1.4.1.1466.115.121.1.41 |
| Protocol Information            | 1.3.6.1.4.1.1466.115.121.1.42 |
| Presentation Address            | 1.3.6.1.4.1.1466.115.121.1.43 | 2.5.5.13
| Printable String                | 1.3.6.1.4.1.1466.115.121.1.44 |
| Subtree Specification           | 1.3.6.1.4.1.1466.115.121.1.45 |
| Supplier Information            | 1.3.6.1.4.1.1466.115.121.1.46 |
| Supplier Or Consumer            | 1.3.6.1.4.1.1466.115.121.1.47 |
| Supplier And Consumer           | 1.3.6.1.4.1.1466.115.121.1.48 |
| Supported Algorithm             | 1.3.6.1.4.1.1466.115.121.1.49 |
| Telephone Number                | 1.3.6.1.4.1.1466.115.121.1.50 |
| Teletex Terminal Identifier     | 1.3.6.1.4.1.1466.115.121.1.51 |
| Telex Number                    | 1.3.6.1.4.1.1466.115.121.1.52 |
| UTC Time                        | 1.3.6.1.4.1.1466.115.121.1.53 | 2.5.5.11
| LDAP Syntax Description         | 1.3.6.1.4.1.1466.115.121.1.54 |
| Modify Rights                   | 1.3.6.1.4.1.1466.115.121.1.55 |
| LDAP Schema Definition          | 1.3.6.1.4.1.1466.115.121.1.56 |
| LDAP Schema Description         | 1.3.6.1.4.1.1466.115.121.1.57 |
| Substring Assertion             | 1.3.6.1.4.1.1466.115.121.1.58 |
| Teletex string with no case sensitivity                       | | 2.5.5.4
| Distinguished name plus a binary large object                 | | 2.5.5.7
| Distinguished Name (DN) string plus a Unicode string (object) | | 2.5.5.14
| Windows New Technology (NT) security descriptor (string)      | | 2.5.5.15
| Security IDentifier (SID) string                              | | 2.5.5.17

https://www.ietf.org/rfc/rfc2252.txt  
https://www.ietf.org/rfc/rfc4517.txt  
</details>

## Kerberos
To setup a Kerberos server using OpenLDAP as backend, follow the instructions in this guide:  
https://ubuntu.com/server/docs/how-to-set-up-kerberos-with-openldap-backend

Now, to allow Kerberos authentication binds against your OpenLDAP, create a Kerberos principal and keytab file for your OpenLDAP server.
```
$ kadmin.local
addprinc -randkey ldap/openldap.example.com
ktadd -k /etc/krb5.keytab ldap/openldap.example.com@EXAMPLE.COM
```

When authenticating via Kerberos (GSSAPI), the username seen by OpenLDAP is in form of `uid=username,cn=example.com,cn=gssapi,cn=auth` instead of the object's DN `cn=username,ou=users,dc=example,dc=com`. This breaks `self` permission ACLs. To solve that, you can configure an identity mapping as described in the [OpenLDAP docs](https://www.openldap.org/doc/admin26/sasl.html).

`ldapmodify -Y EXTERNAL -H ldapi:///`:
```
dn: cn=config
changetype: modify
add: olcAuthzRegexp
olcAuthzRegexp: {0}uid=(.+),cn=gssapi,cn=auth ldap:///dc=example,dc=com??one?(krbPrincipalName:caseIgnoreIA5Match:=$1\40EXAMPLE.COM)
```

Note that `\40` in the LDAP URI represents an escaped `@` char.
