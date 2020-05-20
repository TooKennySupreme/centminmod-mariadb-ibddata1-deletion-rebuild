# How to rebuild MariaDB MySQL InnoDB Database Table Mappings When ibdata1 Is Deleted

`/var/lib/mysql/ibdata1` is an important file which contains MySQL InnoDB database data and mappings for per InnoDB file database tables. If you delete this file and restarted MySQL server, then MySQL server would of re-created a new `ibdata1` file but it will not contain the mapping to your existing InnoDB database tables anymore - you will have broken the mapping and this can result in web applications and MySQL clients not being able to access your MySQL database's InnoDB data. The below guide is a quick overview of how to rebuild this mapping. The guide only works and assumes, you have your `/etc/my.cnf` set with `innodb_file_per_table = 1` to enable per InnoDB table files which will save InnoDB database table data in their own files as opposed to within `ibdata1` itself.

# Grab dbsake and create backup directory 

* Grab [dbsake](https://github.com/abg/dbsake) tool and save to `/usr/bin/dbsake`. Documentation at https://dbsake.readthedocs.io/en/latest/index.html
* Backup directory at /backup-mydata must have enough disk free space to house all your mysql database x2 at least

```
curl -s http://get.dbsake.net > /usr/bin/dbsake
chmod u+x /usr/bin/dbsake
dbsake --version

mkdir -p /backup-mydata
```

# Discard table spaces and import in 5 step stages

Note this guide is for Centmin Mod LEMP stacks which have command shortcut for mysql stop, start and restarts for MariaDB MySQL server and defaults to `innodb_file_per_table = 1` out of the box.

* mysqlstop = service mariadb stop
* mysqlstart = service mariadb start
* mysqlrestart = service mariadb restart

In SSH session populate session variables `newdb` and `olddb` with the new database you will recreate table structure and data from and the original database which you want to fix.

In SSH type the following, which will assign `olddb` = `mysql` database name and `newdb` assigned value of original database name with `new` suffix = mysqlnew.

```
olddb='mysql'
newdb="${olddb}new"
```

Save table listing to file - it may have NULL listings if ibdata1 file was deleted as mapping for database tables is currently lost

```
mysql -e "SET GLOBAL innodb_stats_on_metadata=ON;"

mysql -t -e "SELECT CONCAT(table_schema,'.',table_name) AS 'Table Name', CONCAT(ROUND(table_rows,2),' Rows') AS 'Number of Rows',ENGINE AS 'Storage Engine',CONCAT(ROUND(data_length/(1024*1024),2),'MB') AS 'Data Size',CONCAT(ROUND(index_length/(1024*1024),2),'MB') AS 'Index Size' ,CONCAT(ROUND((data_length+index_length)/(1024*1024),2),'MB') AS'Total', ROW_FORMAT, TABLE_COLLATION FROM information_schema.TABLES WHERE table_schema LIKE '$olddb';" > /backup-mydata/$olddb-tablelistings.txt

cat /backup-mydata/$olddb-tablelistings.txt
```

Use dbsake recreate databases table structures in file named `/backup-mydata/${olddb}_restored.sql`
and then create new ${newdb} database to import `/backup-mydata/${olddb}_restored.sql` to recreate the table structure

```
dbsake frmdump /var/lib/mysql/${olddb}/*.frm > /backup-mydata/${olddb}_restored.sql
mysqladmin create ${newdb}
mysql ${newdb} < /backup-mydata/${olddb}_restored.sql
```

Run 5 step process to DISCARD ${newdb} database table spaces & IMPORT the original database's table data from `.ibd` files

## stage 1

change `olddb` value to the database you want to rebuild from frm files

```
olddb='mysql'
newdb="${olddb}new"

# discard
for tbl in $(ls -1 /var/lib/mysql/${newdb}/*.frm); do
datadir='/var/lib/mysql';
t=$(echo $(basename $tbl) | sed -e 's|.frm||g');
# conversion for hyphenated tablenames
t=$(echo $t | sed -e "s|-|@002d|g")
# only proceed if table's frm file is linked to innodb .ibd file
# otherwise skip for non-innodb tables
if [ -f "$datadir/${olddb}/${t}.ibd" ]; then
echo "DISCARD TABLESPACE for ${newdb}.${t}";
echo "mysql -e \"ALTER TABLE ${newdb}.${t} DISCARD TABLESPACE;\"";
mysql -e "ALTER TABLE ${newdb}.${t} DISCARD TABLESPACE;";
fi
done
```

## stage 2

```
# ibd copy
for tbl in $(ls -1 /var/lib/mysql/${newdb}/*.frm); do
datadir='/var/lib/mysql';
t=$(echo $(basename $tbl) | sed -e 's|.frm||g');
# conversion for hyphenated tablenames
t=$(echo $t | sed -e "s|-|@002d|g")
# only proceed if table's frm file is linked to innodb .ibd file
# otherwise skip for non-innodb tables
if [ -f "$datadir/${olddb}/${t}.ibd" ]; then
echo "COPY $datadir/${olddb}/${t}.ibd" to "$datadir/${newdb}/${t}.ibd";
\cp -af "$datadir/${olddb}/${t}.ibd" "$datadir/${newdb}/${t}.ibd";
chown mysql:mysql "$datadir/${newdb}/${t}.ibd";
fi
done
```

## stage 3

```
# import
for tbl in $(ls -1 /var/lib/mysql/${newdb}/*.frm); do
datadir='/var/lib/mysql';
t=$(echo $(basename $tbl) | sed -e 's|.frm||g');
# conversion for hyphenated tablenames
t=$(echo $t | sed -e "s|-|@002d|g")
# only proceed if table's frm file is linked to innodb .ibd file
# otherwise skip for non-innodb tables
if [ -f "$datadir/${olddb}/${t}.ibd" ]; then
echo "IMPORT TABLESPACE for ${newdb}.${t}";
echo "mysql -e \"ALTER TABLE ${newdb}.${t} IMPORT TABLESPACE;\"";
mysql -e "ALTER TABLE ${newdb}.${t} IMPORT TABLESPACE;";
echo "DROP ${olddb}.${t}";
echo "mysql -e \"DROP TABLE ${t};\" ${olddb}"
mysql -e "DROP TABLE ${t};" ${olddb};
fi
done
```

## stage 4

```
# rename part 1
for tbl in $(ls -1 /var/lib/mysql/${newdb}/*.frm); do
datadir='/var/lib/mysql';
t=$(echo $(basename $tbl) | sed -e 's|.frm||g');
# conversion for hyphenated tablenames
t=$(echo $t | sed -e "s|-|@002d|g")
# only proceed if table's frm file is linked to innodb .ibd file
# otherwise skip for non-innodb tables
if [ -f "$datadir/${olddb}/${t}.ibd" ]; then
echo "rm -f \"$datadir/${olddb}/${t}.ibd\""
rm -f "$datadir/${olddb}/${t}.ibd";
fi
done
```

## stage 5

```
# rename part 2
for tbl in $(ls -1 /var/lib/mysql/${newdb}/*.frm); do
datadir='/var/lib/mysql';
t=$(echo $(basename $tbl) | sed -e 's|.frm||g');
# conversion for hyphenated tablenames
t=$(echo $t | sed -e "s|-|@002d|g")
# only proceed if table's frm file is linked to innodb .ibd file
# otherwise skip for non-innodb tables
if [ -f "$datadir/${newdb}/${t}.ibd" ]; then
echo "mysql -e \"ALTER TABLE ${newdb}.${t} RENAME ${olddb}.${t};\""
mysql -e "ALTER TABLE ${newdb}.${t} RENAME ${olddb}.${t};";
fi
done
```

# Repeat 5 stage steps for all other databases on server

You'll have to repeat the above 5 stage steps for each an every database name on your server listed at /var/lib/mysql data directory changing `olddb` variable to your database name i.e. `olddb=yourdb_name`

For above system `mysql` database it was with

```
olddb='mysql'
newdb="${olddb}new"
```

Where:

**Stage 1** output would be

```
DISCARD TABLESPACE for mysqlnew.gtid_slave_pos
mysql -e "ALTER TABLE mysqlnew.gtid_slave_pos DISCARD TABLESPACE;"
DISCARD TABLESPACE for mysqlnew.innodb_index_stats
mysql -e "ALTER TABLE mysqlnew.innodb_index_stats DISCARD TABLESPACE;"
DISCARD TABLESPACE for mysqlnew.innodb_table_stats
mysql -e "ALTER TABLE mysqlnew.innodb_table_stats DISCARD TABLESPACE;"
DISCARD TABLESPACE for mysqlnew.transaction_registry
mysql -e "ALTER TABLE mysqlnew.transaction_registry DISCARD TABLESPACE;"
```

**Stage 2** output would be

```
COPY /var/lib/mysql/mysql/gtid_slave_pos.ibd to /var/lib/mysql/mysqlnew/gtid_slave_pos.ibd
COPY /var/lib/mysql/mysql/innodb_index_stats.ibd to /var/lib/mysql/mysqlnew/innodb_index_stats.ibd
COPY /var/lib/mysql/mysql/innodb_table_stats.ibd to /var/lib/mysql/mysqlnew/innodb_table_stats.ibd
COPY /var/lib/mysql/mysql/transaction_registry.ibd to /var/lib/mysql/mysqlnew/transaction_registry.ibd
```

**Stage 3** output would be

```
IMPORT TABLESPACE for mysqlnew.gtid_slave_pos
mysql -e "ALTER TABLE mysqlnew.gtid_slave_pos IMPORT TABLESPACE;"
DROP mysql.gtid_slave_pos
mysql -e "DROP TABLE gtid_slave_pos;" mysql
IMPORT TABLESPACE for mysqlnew.innodb_index_stats
mysql -e "ALTER TABLE mysqlnew.innodb_index_stats IMPORT TABLESPACE;"
DROP mysql.innodb_index_stats
mysql -e "DROP TABLE innodb_index_stats;" mysql
IMPORT TABLESPACE for mysqlnew.innodb_table_stats
mysql -e "ALTER TABLE mysqlnew.innodb_table_stats IMPORT TABLESPACE;"
DROP mysql.innodb_table_stats
mysql -e "DROP TABLE innodb_table_stats;" mysql
IMPORT TABLESPACE for mysqlnew.transaction_registry
mysql -e "ALTER TABLE mysqlnew.transaction_registry IMPORT TABLESPACE;"
DROP mysql.transaction_registry
mysql -e "DROP TABLE transaction_registry;" mysql
```

**Stage 4** output would be

```
rm -f "/var/lib/mysql/mysql/gtid_slave_pos.ibd"
rm -f "/var/lib/mysql/mysql/innodb_index_stats.ibd"
rm -f "/var/lib/mysql/mysql/innodb_table_stats.ibd"
rm -f "/var/lib/mysql/mysql/transaction_registry.ibd"
```

**Stage 5** output would be

```
mysql -e "ALTER TABLE mysqlnew.gtid_slave_pos RENAME mysql.gtid_slave_pos;"
mysql -e "ALTER TABLE mysqlnew.innodb_index_stats RENAME mysql.innodb_index_stats;"
mysql -e "ALTER TABLE mysqlnew.innodb_table_stats RENAME mysql.innodb_table_stats;"
mysql -e "ALTER TABLE mysqlnew.transaction_registry RENAME mysql.transaction_registry;"
```

The system `mysql` database has 4 database tables which are InnoDB table based so deleting `ibdata` would have affected the mapping for these 4 database tables

```
| mysql.innodb_table_stats        | NULL           | NULL           | NULL      | NULL       | NULL   | NULL       | NULL            |
| mysql.innodb_index_stats        | NULL           | NULL           | NULL      | NULL       | NULL   | NULL       | NULL            |
| mysql.transaction_registry      | NULL           | NULL           | NULL      | NULL       | NULL   | NULL       | NULL            |
| mysql.gtid_slave_pos            | NULL           | NULL           | NULL      | NULL       | NULL   | NULL       | NULL            |
```

Then recheck restored `mysql` system database

```
mysql -t -e "SELECT CONCAT(table_schema,'.',table_name) AS 'Table Name', CONCAT(ROUND(table_rows,2),' Rows') AS 'Number of Rows',ENGINE AS 'Storage Engine',CONCAT(ROUND(data_length/(1024*1024),2),'MB') AS 'Data Size',CONCAT(ROUND(index_length/(1024*1024),2),'MB') AS 'Index Size' ,CONCAT(ROUND((data_length+index_length)/(1024*1024),2),'MB') AS'Total', ROW_FORMAT, TABLE_COLLATION FROM information_schema.TABLES WHERE table_schema LIKE '$olddb';" > /backup-mydata/$olddb-tablelistings-restored.txt

cat /backup-mydata/$olddb-tablelistings-restored.txt | egrep 'mysql.innodb_table_stats|mysql.innodb_index_stats|mysql.transaction_registry|mysql.gtid_slave_pos'
```
```
cat /backup-mydata/$olddb-tablelistings-restored.txt | egrep 'mysql.innodb_table_stats|mysql.innodb_index_stats|mysql.transaction_registry|mysql.gtid_slave_pos'
| mysql.gtid_slave_pos            | 0 Rows         | InnoDB         | 0.02MB    | 0.00MB     | 0.02MB | Dynamic    | latin1_swedish_ci |
| mysql.innodb_index_stats        | 2186 Rows      | InnoDB         | 0.47MB    | 0.00MB     | 0.47MB | Dynamic    | utf8_bin          |
| mysql.innodb_table_stats        | 230 Rows       | InnoDB         | 0.02MB    | 0.00MB     | 0.02MB | Dynamic    | utf8_bin          |
| mysql.transaction_registry      | 0 Rows         | InnoDB         | 0.02MB    | 0.05MB     | 0.06MB | Dynamic    | utf8_bin          |
```