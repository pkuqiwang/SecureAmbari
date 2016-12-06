# SecureAmbari
This article provide instruction on how to secure Ambari after HDP installation

###Change Ambari to run under non-root user
Ambari runs under root by default after installation. Following instruction change it to run under user ambari-user
On Ambari node, create Unix user ambari-user and grant sudo for ambari
```
useradd -d /var/lib/ambari-server -G hadoop -M -r -s /sbin/nologin ambari-user
echo 'ambari-user ALL=(ALL) NOPASSWD:SETENV: /bin/mkdir, /bin/cp, /bin/chmod, /bin/rm' > /etc/sudoers.d/ambari-server
```
Then run Ambari setup
```
ambari-server stop
ambari-server setup 

OK to continue [y/n] (y)? y
Customize user account for ambari-server daemon [y/n] (n)? y
Enter user account for ambari-server daemon (root):ambari-user
Do you want to change Oracle JDK [y/n] (n)? n
Enter advanced database configuration [y/n] (n)? n
Proceed with configuring remote database connection properties [y/n] (y)? y

ambari-server start
```

###Encrypt password 
Next we will encrypt the passwords stored in ambari.properties file
```
ambari-server stop
ambari-server setup-security

Enter choice, (1-5): 2
Please provide master key for locking the credential store:
Re-enter master key:
Do you want to persist master key. If you choose not to persist, you need to provide the Master Key while starting the ambari server as an env variable named AMBARI_SECURITY_MASTER_KEY or the start will prompt for the master key. Persist [y/n] (y)? y

ambari-server start
```

###Sync LDAP user in Ambari
Next sync the LDAP user to Ambari and set hadoopadmin in LDAP as the admin
```
ambari-server setup-ldap

Primary URL* {host:port} : qwang-kdc-ldap.field.hortonworks.com:389
Secondary URL {host:port} :
Use SSL* [true/false] (false):
User object class* (posixAccount):
User name attribute* (uid):
Group object class* (posixGroup):
Group name attribute* (cn):
Group member attribute* (memberUid):
Distinguished name attribute* (dn):
Base DN* : dc=field,dc=hortonworks,dc=com
Referral method [follow/ignore] :
Bind anonymously* [true/false] (false):
Manager DN* : cn=admin,dc=field,dc=hortonworks,dc=com
Enter Manager Password* :
Re-enter password:
====================
Review Settings
====================
authentication.ldap.managerDn: cn=admin,dc=field,dc=hortonworks,dc=com
authentication.ldap.managerPassword: *****
Save settings [y/n] (y)? y
Saving...done

ambari-server restart
ambari-server sync-ldap --all
ambari-server restart
```
Log into Ambari and turn hadoopadmin user as admin

###Ambari HTTPS
```
openssl req -x509 -newkey rsa:4096 -keyout ambari.key -out ambari.crt -days 1000 -nodes -subj "/CN=qwang-hdp0.field.hortonworks.com"
chown ambari-user ambari.crt ambari.key
chmod 0400 ambari.crt ambari.key
mv ambari.crt /etc/pki/tls/certs/
mv ambari.key /etc/pki/tls/private/
ambari-server setup-security

[1] Enable HTTPS for Ambari server.
Do you want to configure HTTPS [y/n] (y)? y
SSL port [8443] ?
Enter path to Certificate:/etc/pki/tls/certs/ambari.crt 
Enter path to Private Key:/etc/pki/tls/private/ambari.key

ambari-server restart
```
Now the url for Ambari is
```
https://ambari_fqnd:8443
```
You will get a warning will try access the url. Ignore and proceed.
