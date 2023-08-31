# pfSense-LDAP-AuthN
PfSense can use RADIUS and LDAP servers to authenticate users from remote sources. In this example, the firewall connects to an AD structure to authenticate a Security Group.

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/48eaf441-a608-4453-a860-54b4e0099c97"/>
</p> 

On Active Directory, we need a security group and at least two accounts. One account will be used as the service account attempting to bind to the server. Any other account will be a member of the security group that will be able to login on pfSense.

Here are the accounts and group created for this lab:

| Security Group | Description |
| --- | --- |
| ITAdminSG | SG used to login on the pfSense web console  |

| Accounts | Description |
| --- | --- |
| ITUser | Account member of the ITAdminSG security group |
| pfSense-SA | Service account used to establish connection to AD |


With the Active Directory module for Windows PowerShell we can use a group of cmdlets for domain management and managing users, groups, and objects:

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/0ab2258d-83cc-4a5a-8c69-4707333d055b"/>
</p> 

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/da2c6842-d7e9-4e74-8ce5-3bf6779d98f0"/>
</p> 

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/084af259-3dca-41ec-94ec-215c1ee366a1"/>
</p> 

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/edad8d2c-53cb-4e6a-9ce9-b4957c66ecfd"/>
</p> 

Now logging in to the pfSense web console with the local account and password - "admin/pfsense" by default.

System > User Manager > Authentication Servers and click Add:

| Setting | Value |
| --- | --- |
| Descriptive name | AuthN-WINDCLAB01 |
| Type | LDAP |
| Hostname or IP address | 192.168.2.5 |
| Port value | 389 |
| Transport | Standard TCP |
| Peer Certificate Authority | Global Root CA List |
| Protocol version | 3 |
| Server Timeout | 25 |
| Search Scrope | Entire Subtree |
| Base DN | DC=homelab,DC=local |
| Authentication containers | OU=IT,OU=Users,OU=USA,DC=homelab,DC=local |
| Extended query | Checked |
| Query | memberOf=CN=ITAdminSG,OU=IT,OU=Users,OU=USA,DC=homelab,DC=local |
| Bind anonymous | Unchecked |
| Bind credentials | CN=pfSense SA,OU=Service Accounts,OU=Admin,DC=homelab,DC=local |
| Initial Template | Microsoft AD (automatically sets the following 3 values)|
| User naming attribute | samAccountName |
| Group naming attribute | cn |
| Group member attribute | memberOf |
| RFC 2307 Groups | Unchecked |
| Group Object Class | posixGroup |
| Shell Authentication Group DN |  |
| UTF8 Encode | Unchecked |
| Username Alterations | Unchecked |
| Allow unauthenticated bind | Unchecked |


<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/9a188fd9-c683-4565-8435-e0b5ce5ecea0"/>
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/514a36a6-e3a0-48de-b0fe-d628ed6962a1"/>
</p>


Before moving forward, we can test these configurations by clicking "Select a container", a list of containers should appear, which means that the firewall crafted LDAP queries. Otherwise, we will get an error such as "Could not connect to the LDAP server. Please check the LDAP configuration."

Additional an AD authentication test can be done:

Diagnostics > Authentication > Provide the previously configured Authentication Server (AuthN-WINDCLAB01) and an AD user (ITUser) that is a member of the SG (ITAdminSG) configured in the Query.

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/9d6c60e9-ec50-4283-88de-5935df610cf6"/>
</p>


