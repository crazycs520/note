# install

## on mac

```shell
brew install mysql	
```

config the mysql database

```shell
unset TMPDIR

mysql_install_db --verbose --user=`dbUserName` --basedir="$(brew --prefix mysql)" --datadir=/usr/local/var/mysql --tmpdir=/tmp

```



start the mysql server

```Shell
mysql.server start
```

connect to mysql

```Shell
mysql -u root
```

