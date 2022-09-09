# galaxy-ftp
Instructions to install proftpd for galaxy uploads

https://docs.galaxyproject.org/en/master/admin/special_topics/ftp.html

Get postgresql version

```
[postgres@vclvm178-23 etc]$ psql
psql (13.6)
```


Find installed verson and get location of posgresql libraries and includes
```
rpm -qa | grep postgres
rpm -ql postgresql13-devel-13.6
```

get proftpd code
configure --enable-ident can be used for testing 

```
wget https://github.com/proftpd/proftpd/archive/refs/tags/v1.3.7e.tar.gz

./configure  --enable-openssl --with-modules=mod_sql:mod_sql_postgres:mod_sql_passwd:mod_tls --with-includes=/usr/pgsql-13/include:/usr/include/openssl --with-libraries=/usr/pgsql-13/lib:/usr/lib64/ --with-postgres-config=/usr/pgsql-13/bin
make
sudo make install

```
Get ids for user to run ftp server. 

```
grep galaxy /etc/passwd
galaxy:x:58413:58413::/home/galaxy:/bin/bash

```

```
[postgres@vclvm178-23 jbwhite2]$ createuser -SDR galaxyftp
[postgres@vclvm178-23 jbwhite2]$ psql
psql (13.6)
Type "help" for help.

postgres=# \c galaxydb 
You are now connected to database "galaxydb" as user "postgres".
galaxydb=# ALTER ROLE galaxyftp PASSWORD 'ftppassword';
ALTER ROLE
galaxydb=# GRANT SELECT ON galaxy_user TO galaxyftp; 
GRANT
galaxydb=# 

```
Copy config to /usr/local/etc/proftpd.conf

```
# Basics, some site-specific
ServerName                      "Public Galaxy FTP"
ServerType                      standalone
DefaultServer                   on
Port                            21
Umask                           077
SyslogFacility                  DAEMON
SyslogLevel                     debug
MaxInstances                    30

# This User & Group should be set to the actual user and group name which matche the UID & GID you will specify later in the SQLNamedQuery.
User                            galaxy
Group                           galaxy
DisplayConnect                  /etc/local/proftpd/welcome.txt

# Passive port range for the firewall
PassivePorts                    30000 40000

# Cause every FTP user to be "jailed" (chrooted) into their home directory
DefaultRoot                     ~

# Automatically create home directory if it doesn't exist
CreateHome                      on dirmode 700

# Allow users to overwrite their files
AllowOverwrite                  on

# Allow users to resume interrupted uploads
AllowStoreRestart               on

# Bar use of SITE CHMOD
<Limit SITE_CHMOD>
    DenyAll
</Limit>

# Bar use of RETR (download) since this is not a public file drop
<Limit RETR>
    DenyAll
</Limit>

# Do not authenticate against real (system) users
<IfModule mod_auth_pam.c>
AuthPAM                         off
</IfModule>

<IfModule mod_tls.c>
    TLSEngine on
    # TLSLog /var/ftpd/tls.log

    # Support TLSv1, TLSv1.1, and TLSv1.2
    TLSProtocol TLSv1 TLSv1.1 TLSv1.2

    # Are clients required to use FTP over TLS when talking to this server?
    TLSRequired off



    # Server's RSA certificate
     TLSRSACertificateFile /etc/letsencrypt/live/vclvm178-23.vcl.ncsu.edu/cert.pem
     TLSRSACertificateKeyFile /etc/letsencrypt/live/vclvm178-23.vcl.ncsu.edu/privkey.pem


    # Authenticate clients that want to use FTP over TLS?
    TLSVerifyClient off

    # Allow SSL/TLS renegotiations when the client requests them, but
    # do not force the renegotiations.  Some clients do not support
    # SSL/TLS renegotiations; when mod_tls forces a renegotiation, these
    # clients will close the data connection, or there will be a timeout
    # on an idle data connection.
    TLSRenegotiate none

</IfModule>

# Common SQL authentication options
SQLEngine                       on
SQLPasswordEngine               on
SQLBackend                      postgres
SQLConnectInfo                  galaxydb@localhost:5432 galaxyftp ftppassword
SQLAuthenticate                 users

# Set up mod_sql/mod_sql_password - Galaxy passwords are stored as hex-encoded SHA1
SQLAuthTypes                    SHA1
SQLPasswordEncoding             hex


# Define a custom query for lookup that returns a passwd-like entry. Replace 512s with the UID and GID of the user running the Galaxy server
SQLUserInfo                     custom:/LookupGalaxyUser
SQLNamedQuery                   LookupGalaxyUser SELECT "email,password,58413,58413,'/icarbon_temp_10tb/database/files/%U','/bin/bash' FROM galaxy_user WHERE email='%U'"


```

