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

```
wget https://github.com/proftpd/proftpd/archive/refs/tags/v1.3.7e.tar.gz

./configure --disable-auth-file --disable-ncurses --disable-ident --disable-shadow --enable-openssl --with-modules=mod_sql:mod_sql_postgres:mod_sql_passwd --with-includes=/usr/pgsql-13/include:`pwd`/../openssl/.openssl/include --with-libraries=/usr/pgsql-13/lib:`pwd`/../openssl/.openssl/lib --with-postgres-config=/usr/pgsql-13/bin
```
