# pfSense-LDAP-AuthN
PfSense can use RADIUS and LDAP servers to authenticate users from remote sources. In this example, the pfSense firewall connects to a Windows Domain Controller to authenticate an AD Security Group.

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/48eaf441-a608-4453-a860-54b4e0099c97"/>
</p> 


| Virtual Machine | IP Address | Description |
| --- | --- | --- |
| WINPCLAB01 | 192.168.1.5 | Windows Client where pfSense is accessed from |
| WINDCLAB01 | 192.168.2.5 | Windows Domain Controller |


On Active Directory, we need a security group and at least two accounts. One account will be used as the service account attempting to bind to the server. Any other account will be a member of the security group that will be able to login on pfSense.

Here are the accounts and group already created in AD:

| Security Group | Description |
| --- | --- |
| ITAdminSG | SG to login on the pfSense web console |

| Accounts | Description |
| --- | --- |
| ITUser | Account member of the ITAdminSG security group |
| pfSense-SA | Service account to establish connection to AD |



With the Active Directory module for Windows PowerShell we can use a group of cmdlets to manage domain, users, groups, and objects:

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

pfSense
<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/9a188fd9-c683-4565-8435-e0b5ce5ecea0"/>
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/514a36a6-e3a0-48de-b0fe-d628ed6962a1"/>
</p>


Before moving forward, we can test these configurations by clicking "Select a container", a list of containers should appear, which means that the firewall crafted LDAP queries. Otherwise, we will get an error such as "Could not connect to the LDAP server. Please check the LDAP configuration."

Additional an AD authentication test can be performed as follows:

Diagnostics > Authentication > Provide the previously configured Authentication Server (AuthN-WINDCLAB01) and an AD user (ITUser) that is a member of the SG (ITAdminSG) configured in the Query:

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/9d6c60e9-ec50-4283-88de-5935df610cf6"/>
</p>

Even though it says ITUser authenticated successfully, it cannot log in because first, ITUser doesn't seem to be a member of a group yet, and second, the Authentication Server is still disabled.

When working with group privileges and authentication servers, there must be local groups with names that exactly match the groups in AD:

| Active Directory | pfSense |
| --- | --- |
| ITAdminSG | ITAdminSG |

System > User Manager > Groups and click Add:

| Setting | Value |
| --- | --- |
| Group name | ITAdminSG |
| Scope  | Remote |
| Description | AD Users |

pfSense
<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/2e06193d-4f85-4571-9cdf-451f1fa387e2"/>
</p>

Click Save, then it will return to previous Group screen:

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/f21217e5-f517-4911-b138-5e2021e733d4"/>
</p>

Click the Edit icon for the ITAdminSG group and then Add to assign privileges and finally Save.

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/6e52ab73-2462-4f0d-8d97-486365e59d7f"/>
</p>

Going back to Diagnostics > Authentication > Provide the previously configured Authentication Server (AuthN-WINDCLAB01) and an AD user (ITUser) that is a member of the SG (ITAdminSG) configured in the Query:

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/5af18759-a909-4736-93dd-0199a0d335ee"/>
</p>

Now we can move forward and enable the Authentication Server.

System > User Manager > Settings > Select the Authentication Server (AuthN-WINDCLAB01) and click Save & Test:

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/6113bc86-1631-48c4-b9c2-2b6067d44ec0"/>
   <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/5da4a008-01c8-4a42-9b07-6af3f97c4f7f"/>
</p>

Finally we are able to log in to the pfSense web console with ITUser account or any other account in the ITAdminSG security group:

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/2be2b249-838c-4855-aa3a-dabc87c38c12"/>
  
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/718fdffe-5be0-4460-afce-2d706de1bab6"/>

  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/2f13a900-ee23-45fd-aac0-2aef1effd051"/>
</p>


