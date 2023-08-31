# pfSense-LDAP-AuthN
PfSense can use RADIUS and LDAP servers to authenticate users from remote sources. In this example, the pfSense firewall connects to a Windows Domain Controller to authenticate an AD Security Group.

<p align="center">
 <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/7f96e62a-c182-4a10-b7e7-95234b61b8ac"/>
</p> 


| Virtual Machine | IP Address | Description |
| --- | --- | --- |
| WINPCLAB01 | 192.168.1.5 | Windows Client where pfSense is accessed from |
| WINDCLAB01 | 192.168.2.5 | Windows Domain Controller |


On Active Directory, we need a security group and at least two accounts. One account will be used as the service account attempting to bind to the server. Any other account will be a member of the security group that will be able to login on pfSense.

Here are the accounts and a group already created in AD:

| Security Group | Description |
| --- | --- |
| ITAdminSG | SG to log in on the pfSense web console |

| Accounts | Description |
| --- | --- |
| ITUser | Account member of the ITAdminSG security group |
| pfSense-SA | Service account to establish a connection to AD |



With the Active Directory module for Windows PowerShell, we can use a group of cmdlets to manage domains, users, groups, and objects:

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/3669ff98-e5f0-4027-ba81-8124fa74125b"/>
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/6414e937-0ea0-4f72-9eea-d296f19e170b"/>
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/ed53ff46-13cc-473b-a0be-f52daf8ad7f8)"/>
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/4ddf1c5f-55fd-4927-8538-4d84d58ce0cd)"/>
</p> 

Now log in to the pfSense web console with the local account and password - "admin/pfsense" by default.

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
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/571fcf7b-055a-4c1a-8fde-718c27e7de95"/>
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/389fac34-697d-4d67-9a4d-fc2b9ab5309f"/>
</p>


Before moving forward, we can test these configurations by clicking "Select a container". A list of containers should appear, which means that the firewall crafted LDAP queries. Otherwise, we will get an error such as "Could not connect to the LDAP server. Please check the LDAP configuration."

Additionally, an AD authentication test can be performed as follows:

Diagnostics > Authentication > Provide the previously configured Authentication Server (AuthN-WINDCLAB01) and an AD user (ITUser) that is a member of the SG (ITAdminSG) configured in the Query:

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/59fe03b2-31d8-412b-8f1a-628554005f6d"/>
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
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/7ee1aa1d-56a7-4e02-a56f-d29457b25827"/>
</p>

Click Save, then it will return to the previous Group screen:

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/3564af30-a07a-4a58-887e-8def3c31be7c"/>
</p>

Click the Edit icon for the ITAdminSG group then Add to assign privileges and finally Save.

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/41be39ea-6010-416c-96c3-a9460ce296d2"/>
</p>

Going back to Diagnostics > Authentication > Provide the previously configured Authentication Server (AuthN-WINDCLAB01) and an AD user (ITUser) that is a member of the SG (ITAdminSG) configured in the Query:

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/04ce2f5a-728e-4ab4-bc25-c3158041c6a0"/>
</p>

Now we can move forward and enable the Authentication Server.

System > User Manager > Settings > Select the Authentication Server (AuthN-WINDCLAB01) and click Save & Test:

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/81d16e1f-205f-4fa5-8575-76ddf196da9d"/>
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/c86865c1-ebf9-4229-b64d-b1899e5af2c4"/>
</p>

Finally, we are able to log in to the pfSense web console with the ITUser account or any other account in the ITAdminSG security group:

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/ebee50a5-ca76-4e19-a28d-aae7dd0e3cea"/>
  
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/efcc9a3a-435b-48d4-b538-d1dfdd1fffe6"/>

  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/67955429-9d4f-41c8-96b1-65691ecacfcb"/>
</p>


