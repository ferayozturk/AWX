# LDAP Configuration for AWX

### 1) First of all, you should check the access from server installed AWX to Ldap server.
The IP address of AWX server -> 10.10.202.61
The URL of LDAP server -> ldap2.mydomain.com

```sh
[root@istpawxv installer]# curl -v telnet://ldap2.mydomain.com:389
* Rebuilt URL to: telnet://ldap2.mydomain.com:389/
*   Trying ...
* TCP_NODELAY set
* Connected to ldap2.mydomain.com port 389 (#0)
```

