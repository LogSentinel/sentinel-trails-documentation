LDAP & Active Directory
=======================

SentienlTrails supports integration with LDAP and ActiveDirectory for authentication. Below is a set of steps to be executed in order to configure the integration:

LDAP configuration
******************

In order to authenticate with LDAP server (not Windows AD) the following steps must be done:

* in ``application.properties`` set ``ldap.auth.enabled=true`` and ``ad.auth.enabled=false``
* set ``ldap.url` to pint to your LDAP server
* let's assume the following ldap tree:

.. code:: text

	dn: dc=yourOrganization,dc=org
	objectclass: top
	objectclass: domain
	objectclass: extensibleObject
	dc: ourOrganization

	dn: ou=groups,dc=springframework,dc=org
	objectclass: top
	objectclass: organizationalUnit
	ou: groups

	dn: ou=people,dc=springframework,dc=org
	objectclass: top
	objectclass: organizationalUnit
	ou: people

	dn: uid=ben@yourOrganization.org,ou=people,dc=yourOrganization,dc=org
	objectclass: top
	objectclass: person
	objectclass: organizationalPerson
	objectclass: inetOrgPerson
	cn: Ben Alexx
	sn: Alex
	organizationId: ba2cbc90-5424-11e8-b88d-6f2c1b6625e8
	uid: ben@yourOrganization.org
	userPassword: {SHA}nFCebWjxfaLbHHG1Qk5UU4trbvQ=

	dn: cn=logsentinel_developer,ou=groups,dc=springframework,dc=org
	objectclass: top
	objectclass: groupOfUniqueNames
	cn: logsentinel_developer
	ou: logsentinel_developer
	uniqueMember: uid=ben@yourOrganization.org,ou=people,dc=yourOrganization,dc=org


* if ldap server doesn't give read access to annonymos users ldap.manager.DN and ldap.manager.password should be set
* user is searched with ldap.userDN.pattern=uid={0},ou=people  -> ben@yourOrganization.org
* gropus are searched with ldap.group.searchbase=ou=groups
* encrypted password pattern : ldap.password.attribute=userPassword
* Names of the user are axtracted by ldap.names.param=cn
* Organization id of the user (in logsentinel) can be assigned with the default value ``ldap.default.organizationId``. It can also be retrieved from LDAP if it is set to the user as an additional property ``ldap.organizationId.param=organizationId``
* user role is retrieved from the groups that user is included. Only groups starting with ``logsentinel_`` are taken into account. For example here ``ben@yourOrganization.org`` is member of the group ``logsentinel_developer``, so the retrieved role will be DEVELOPER. There are 5 acceptable roles: DEVELOPER, MANAGER, AUDITOR, ADMIN, IT


Active Directory configuration
******************************

In order to authenticate with Windows Active Directory:

* set ``ad.auth.enabled=true`` and ``ldap.auth.enabled=false`` in ``application.properties``
* set ``ldap.url`` to the URL of the Active Directory (without the dc parts)
* set ``ad.auth.domain`` to the domain where users belong in AD
* roles are retrieved from the ``memberOf`` attributes of the user in AD. Same roles restrictions as in LDAP apply.

