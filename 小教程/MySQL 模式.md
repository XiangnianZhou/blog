1. `sudo vim /etc/mysql/my.cnf`
2. 在文件底部粘贴下面代码：
  ```
[mysqld]
sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION'
  ```
3. 保存退出 vim
4. 重启 MySQL： `sudo service mysql restart`