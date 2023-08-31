# pfSense-LDAP-AuthN
PfSense can use RADIUS and LDAP servers to authenticate users from remote sources. In this example, the firewall connects to an AD structure to authenticate a Security Group.

<p align="center">
  <img src="https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/48eaf441-a608-4453-a860-54b4e0099c97"/>
</p> 

On Active Directory, we need a security group and at least two accounts. One account will be used as the service account attempting to bind to the server. Any other account will be a member of the security group that will be able to login on pfSense.

Here are the accounts and group created for this lab:

| Security Group | Description |
| --- | --- |
| ITAdminSG | SG used to login on the pfSense web interface  |

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
  <img src=https://github.com/HamllerM/pfSense-LDAP-AuthN/assets/62651116/edad8d2c-53cb-4e6a-9ce9-b4957c66ecfd/>
</p> 

