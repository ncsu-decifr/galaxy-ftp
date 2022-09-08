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
find in galaxy.yml for conneciton info

database_connection: 'postgresql://galaxy:mypassword@127.0.0.1/galaxydb'

login as user runnig galaxy and get users id and group id 

```
id -u
id -g

```

