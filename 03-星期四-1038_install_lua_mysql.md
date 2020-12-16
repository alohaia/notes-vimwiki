# 03-星期四-1038 install lua mysql

- lua-mysql

```bash
sudo proxychains -q luarocks install luasql-mysql MYSQL_INCDIR=/usr/include/mysql
```
- [MariaDB](MariaDB.md)

```bash
sudo pacman -S mariadb mariadb-clients mariadb-libs
sudo mv /var/lib/mysql /tmp
sudo mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
sudo systemctl enable mariadb
sudo systemctl restart mariadb
mysql
```
