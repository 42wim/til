# Password expiration

An application started to fail with this error.

```
Feb 06 07:00:01 docker-current[1799]: 2017-02-06T06:00:01.954900Z 6322689 [Note] Your password has expired. To log in you must change it using a client that supports expired passwords.
```

This happened because the mysql we were running apparently has automatic password expiration.

Mysql 5.7.4 - 5.7.10 has automatic password expiration (after 360 days)
<https://dev.mysql.com/doc/refman/5.7/en/password-expiration-policy.html>

Luckily this was fixed in
<https://dev.mysql.com/doc/relnotes/mysql/5.7/en/news-5-7-11.html>

The default value of the default_password_lifetime system variable that controls the global password expiration policy has been changed from 360 (360 days) to 0 (no password expiration). The default of 360 **sometimes took people by surprise when account passwords expired a year** after upgrading to MySQL 5.7. 

But to fix it if still running Mysql 5.7.4 - 5.7.10

```
mysql -uroot -p --connect-expired-password
SET GLOBAL default_password_lifetime = 0;
```
### More information
<http://dba.stackexchange.com/questions/118852/your-paswssword-has-expired-after-restart-mysql-when-updated-mysql-5-7-8-rcde>
<http://mysqlblog.fivefarmers.com/2014/03/31/password-expiration-policy-in-mysql-server-5-7/>
