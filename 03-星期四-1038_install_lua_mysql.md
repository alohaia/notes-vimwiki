# 03-星期四-1038 install lua mysql

- lua-mysql

```bash
sudo proxychains -q luarocks install luasql-mysql MYSQL_INCDIR=/usr/include/mysql
```
- MariaDB

```bash
sudo pacman -S mariadb mariadb-clients mariadb-libs
sudo mv /var/lib/mysql /tmp
mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
sudo mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
reboot
mysql
```
