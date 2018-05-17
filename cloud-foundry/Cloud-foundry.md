# Cloud foundry

## 杂项

- connect to ccdb:

  在 `cf-deployment-var.yml` 里找到 `cf_mysql_mysql_admin_password` ，用 bosh ssh 到 database ， 在命令行里执行 `mysql -S /var/vcap/sys/run/mysql/mysqld.sock -u root -p <cf_mysql_mysql_admin_password>` 即可。
