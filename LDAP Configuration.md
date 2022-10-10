# LDAP Configuration for AWX

### First of all, you should check the access from server installed AWX to Ldap server.
The IP address of AWX server -> 10.10.202.61
The URL of LDAP server -> ldap.mydomain.com

```sh
[root@istpawxv installer]# curl -v telnet://ldap.mydomain.com:389
* Rebuilt URL to: telnet://ldap.mydomain.com:389/
*   Trying ...
* TCP_NODELAY set
* Connected to ldap.mydomain.com port 389 (#0)
```

Now, you are ready to do ldap configuration.

### Login to AWX console 
After you logined to the console, choose "Settings" from the left menu. And then you should choose "LDAP settings".

<img width="361" alt="Capture" src="https://user-images.githubusercontent.com/28459280/194851207-0f267786-4bcc-4b6e-8b14-710ace2504a9.PNG">

### Edittin default LDAP configuration

- LDAP Server URL: ldap://ldap.mydomain.com:389
- LDAP Bind Password: *********** #you should use the password of the service_user connecting to ldap. 
- LDAP Group Type: MemberDNGroupType
- LDAP Bind DN: CN=<service_user>,OU=<ou name>,DC=<domain name>,DC=<top level domain>
- LDAP User DNS Template: <blank>
- LDAP Required Group: CN=<group_name>,DC=<domain name>,DC=<top level domain>  #If you only want members of your group to log in, you must add the group name here. Thus, only members of this group can log in.
- LDAP Deny Group: <blank>
- LDAP User Search:
  ```sh
  [
  "DC=<domain name>,DC=<domain name>,DC=com",
  "SCOPE_SUBTREE",
  "(sAMAccountName=%(user)s)"
]
  ```
  
 - LDAP Group Search:
```sh
  [
  "DC=<domain name>,DC=<domain name>,DC=com",
  "SCOPE_SUBTREE",
  "(objectClass=group)"
]
  ```
  
 - LDAP User Attribute Map:
```sh
{
  "first_name": "givenName",
  "last_name": "sn",
  "email": "mail"
}
  ```
   
  - LDAP Group Type Parameters:
  ```sh
{
  "name_attr": "cn",
  "member_attr": "member"
}
  ```
  
  - LDAP User Flags By Group: <blank>
  - LDAP Organization Map: <blank>
  - LDAP Team Map: <blank>
  
