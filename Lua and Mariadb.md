# 02-星期三-1218-Lua and Mariadb

```bash
sudo pacman -S mariadb mariadb-libs
mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```

> 如果数据目录使用的不是 /var/lib/mysql，需要在 /etc/my.cnf.d/server.cnf 文件的 [mysqld] 部分设置 datadir=<数据目录>
