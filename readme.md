# How to rebuild MariaDB MySQL InnoDB Database Table Mappings When ibdata1 Is Deleted

`/var/lib/mysql/ibdata1` is an important file which contains MySQL InnoDB database data and mappings for per InnoDB file database tables. If you delete this file and restarted MySQL server, then MySQL server would of re-created a new `ibdata1` file but it will not contain the mapping to your existing InnoDB database tables anymore - you will have broken the mapping and this can result in web applications and MySQL clients not being able to access your MySQL database's InnoDB data. The below guide is a quick overview of how to rebuild this mapping. The guide only works and assumes, you have your `/etc/my.cnf` set with `innodb_file_per_table = 1` to enable per InnoDB table files which will save InnoDB database table data in their own files as opposed to within `ibdata1` itself.

If `/var/lib/mysql/ibdata1` is deleted, then MySQL server would re-created a new `ibdata1` file on MySQL server startup but you will loose MySQL InnoDB database/table mapping

```
journalctl -u mariadb --no-pager | tail -35
May 20 21:10:45 host systemd[1]: Starting MariaDB 10.3.23 database server...
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] /usr/sbin/mysqld (mysqld 10.3.23-MariaDB) starting as process 2720 ...
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Using Linux native AIO
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: The first innodb_system data file 'ibdata1' did not exist. A new tablespace will be created!
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Uses event mutexes
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Compressed tables use zlib 1.2.7
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Number of pools: 1
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Using SSE2 crc32 instructions
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Initializing buffer pool, total size = 48M, instances = 1, chunk size = 48M
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Completed initialization of buffer pool
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: If the mysqld execution user is authorized, page cleaner thread priority can be changed. See the man page of setpriority().
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Setting file './ibdata1' size to 10 MB. Physically writing the file full; Please wait ...
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: File './ibdata1' size is now 10 MB.
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Setting log file ./ib_logfile101 size to 134217728 bytes
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Setting log file ./ib_logfile1 size to 134217728 bytes
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Renaming log file ./ib_logfile101 to ./ib_logfile0
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: New log files created, LSN=44898
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Doublewrite buffer not found: creating new
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Doublewrite buffer created
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: 128 out of 128 rollback segments are active.
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Creating foreign key constraint system tables.
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Creating tablespace and datafile system tables.
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Creating sys_virtual system tables.
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Creating shared tablespace for temporary tables
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: File './ibtmp1' size is now 12 MB.
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: Waiting for purge to start
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: 10.3.23 started; log sequence number 0; transaction id 7
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] Plugin 'FEEDBACK' is disabled.
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] Server socket created on IP: '::'.
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 3 [Warning] Failed to load slave replication state from table mysql.gtid_slave_pos: 1932: Table 'mysql.gtid_slave_pos' doesn't exist in engine
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] /usr/sbin/mysqld: ready for connections.
May 20 21:10:46 host mysqld[2720]: Version: '10.3.23-MariaDB'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  MariaDB Server
May 20 21:10:46 host systemd[1]: Started MariaDB 10.3.23 database server.
```

Notice some of the signs in the logging for messages such as

```
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 0 [Note] InnoDB: The first innodb_system data file 'ibdata1' did not exist. A new tablespace will be created!
```

and missing system `mysql` database tables which are there physically but lost it's mapping due to `ibdata1` file being deleted.

```
May 20 21:10:46 host mysqld[2720]: 2020-05-20 21:10:46 3 [Warning] Failed to load slave replication state from table mysql.gtid_slave_pos: 1932: Table 'mysql.gtid_slave_pos' doesn't exist in engine
```

Inspecting system `mysql` database tables show the InnoDB tables for `mysql.gtid_slave_pos`, `mysql.innodb_index_stats`, `mysql.innodb_table_stats` and `mysql.transaction_registry` are reporting NULL when `ibdata1` was deleted while other system tables are fine (because they're MyISAM based and not affected by `ibdata1` deletion)

```
mysql -t -e "SELECT CONCAT(table_schema,'.',table_name) AS 'Table Name', CONCAT(ROUND(table_rows,2),' Rows') AS 'Number of Rows',ENGINE AS 'Storage Engine',CONCAT(ROUND(data_length/(1024*1024),2),'MB') AS 'Data Size',CONCAT(ROUND(index_length/(1024*1024),2),'MB') AS 'Index Size' ,CONCAT(ROUND((data_length+index_length)/(1024*1024),2),'MB') AS'Total', ROW_FORMAT, TABLE_COLLATION FROM information_schema.TABLES WHERE table_schema LIKE '$olddb';"
+---------------------------------+----------------+----------------+-----------+------------+--------+------------+-----------------+
| Table Name                      | Number of Rows | Storage Engine | Data Size | Index Size | Total  | ROW_FORMAT | TABLE_COLLATION |
+---------------------------------+----------------+----------------+-----------+------------+--------+------------+-----------------+
| mysql.plugin                    | 4 Rows         | MyISAM         | 0.00MB    | 0.00MB     | 0.00MB | Dynamic    | utf8_general_ci |
| mysql.tables_priv               | 0 Rows         | MyISAM         | 0.00MB    | 0.00MB     | 0.00MB | Fixed      | utf8_bin        |
| mysql.roles_mapping             | 0 Rows         | MyISAM         | 0.00MB    | 0.00MB     | 0.00MB | Fixed      | utf8_bin        |
| mysql.user                      | 5 Rows         | MyISAM         | 0.00MB    | 0.00MB     | 0.00MB | Dynamic    | utf8_bin        |
| mysql.servers                   | 0 Rows         | MyISAM         | 0.00MB    | 0.00MB     | 0.00MB | Fixed      | utf8_general_ci |
| mysql.columns_priv              | 0 Rows         | MyISAM         | 0.00MB    | 0.00MB     | 0.00MB | Fixed      | utf8_bin        |
| mysql.db                        | 3 Rows         | MyISAM         | 0.00MB    | 0.01MB     | 0.01MB | Fixed      | utf8_bin        |
| mysql.help_topic                | 508 Rows       | MyISAM         | 0.39MB    | 0.02MB     | 0.41MB | Dynamic    | utf8_general_ci |
| mysql.help_category             | 39 Rows        | MyISAM         | 0.00MB    | 0.00MB     | 0.00MB | Dynamic    | utf8_general_ci |
| mysql.help_relation             | 1028 Rows      | MyISAM         | 0.01MB    | 0.02MB     | 0.03MB | Fixed      | utf8_general_ci |
| mysql.help_keyword              | 464 Rows       | MyISAM         | 0.09MB    | 0.02MB     | 0.10MB | Fixed      | utf8_general_ci |
| mysql.time_zone_name            | 0 Rows         | MyISAM         | 0.00MB    | 0.00MB     | 0.00MB | Fixed      | utf8_general_ci |
| mysql.time_zone                 | 0 Rows         | MyISAM         | 0.00MB    | 0.00MB     | 0.00MB | Fixed      | utf8_general_ci |
| mysql.time_zone_transition      | 0 Rows         | MyISAM         | 0.00MB    | 0.00MB     | 0.00MB | Fixed      | utf8_general_ci |
| mysql.time_zone_transition_type | 0 Rows         | MyISAM         | 0.00MB    | 0.00MB     | 0.00MB | Fixed      | utf8_general_ci |
| mysql.time_zone_leap_second     | 0 Rows         | MyISAM         | 0.00MB    | 0.00MB     | 0.00MB | Fixed      | utf8_general_ci |
| mysql.proc                      | 2 Rows         | MyISAM         | 0.00MB    | 0.00MB     | 0.01MB | Dynamic    | utf8_general_ci |
| mysql.host                      | 0 Rows         | MyISAM         | 0.00MB    | 0.00MB     | 0.00MB | Fixed      | utf8_bin        |
| mysql.general_log               | 2 Rows         | CSV            | 0.00MB    | 0.00MB     | 0.00MB | Dynamic    | utf8_general_ci |
| mysql.slow_log                  | 2 Rows         | CSV            | 0.00MB    | 0.00MB     | 0.00MB | Dynamic    | utf8_general_ci |
| mysql.event                     | 0 Rows         | MyISAM         | 0.00MB    | 0.00MB     | 0.00MB | Dynamic    | utf8_general_ci |
| mysql.gtid_slave_pos            | NULL           | NULL           | NULL      | NULL       | NULL   | NULL       | NULL            |
| mysql.innodb_index_stats        | NULL           | NULL           | NULL      | NULL       | NULL   | NULL       | NULL            |
| mysql.proxies_priv              | 2 Rows         | MyISAM         | 0.00MB    | 0.01MB     | 0.01MB | Fixed      | utf8_bin        |
| mysql.table_stats               | 0 Rows         | MyISAM         | 0.00MB    | 0.00MB     | 0.00MB | Dynamic    | utf8_bin        |
| mysql.column_stats              | 0 Rows         | MyISAM         | 0.00MB    | 0.00MB     | 0.00MB | Dynamic    | utf8_bin        |
| mysql.index_stats               | 0 Rows         | MyISAM         | 0.00MB    | 0.00MB     | 0.00MB | Dynamic    | utf8_bin        |
| mysql.innodb_table_stats        | NULL           | NULL           | NULL      | NULL       | NULL   | NULL       | NULL            |
| mysql.transaction_registry      | NULL           | NULL           | NULL      | NULL       | NULL   | NULL       | NULL            |
| mysql.func                      | 0 Rows         | MyISAM         | 0.00MB    | 0.00MB     | 0.00MB | Fixed      | utf8_bin        |
| mysql.procs_priv                | 0 Rows         | MyISAM         | 0.00MB    | 0.00MB     | 0.00MB | Fixed      | utf8_bin        |
+---------------------------------+----------------+----------------+-----------+------------+--------+------------+-----------------+
```

It's as if InnoDB database table doesn't exist when `ibdata1` was deleted

```
mysql -e "SELECT * FROM innodb_table_stats;" mysql
ERROR 1932 (42S02) at line 1: Table 'mysql.innodb_table_stats' doesn't exist in engine
```

But they physically do exist, just the mapping was lost with deletion of `ibdata1`.

```
ls -lAhrt /var/lib/mysql/mysql | egrep 'innodb_table_stats|innodb_index_stats|transaction_registry|gtid_slave_pos'
-rw-rw----  1 mysql mysql 1.0K May 20 08:41 gtid_slave_pos.frm
-rw-rw----  1 mysql mysql 5.3K May 20 08:41 innodb_index_stats.frm
-rw-rw----  1 mysql mysql 1.9K May 20 08:41 innodb_table_stats.frm
-rw-rw----  1 mysql mysql 2.6K May 20 08:41 transaction_registry.frm
-rw-------. 1 mysql mysql  96K May 20 08:42 gtid_slave_pos.ibd
-rw-------. 1 mysql mysql 9.0M May 20 08:42 innodb_index_stats.ibd
-rw-------. 1 mysql mysql 128K May 20 08:42 innodb_table_stats.ibd
-rw-------. 1 mysql mysql 144K May 20 08:42 transaction_registry.ibd
```

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

Run 5 step process to DISCARD ${newdb} database table spaces & IMPORT the original database's table data from `.ibd` files. Each of the 5 step stages are a for loop so you need to copy and paste the entire `for ... done` loop lines into SSH command line for it to work.

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
tc=$(echo $t | sed -e "s|-|@002d|g")
# only proceed if table's frm file is linked to innodb .ibd file
# otherwise skip for non-innodb tables
if [ -f "$datadir/${olddb}/${tc}.ibd" ]; then
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
tc=$(echo $t | sed -e "s|-|@002d|g")
# only proceed if table's frm file is linked to innodb .ibd file
# otherwise skip for non-innodb tables
if [ -f "$datadir/${olddb}/${tc}.ibd" ]; then
echo "COPY $datadir/${olddb}/${tc}.ibd" to "$datadir/${newdb}/${tc}.ibd";
\cp -af "$datadir/${olddb}/${tc}.ibd" "$datadir/${newdb}/${tc}.ibd";
chown mysql:mysql "$datadir/${newdb}/${tc}.ibd";
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
tc=$(echo $t | sed -e "s|-|@002d|g")
# only proceed if table's frm file is linked to innodb .ibd file
# otherwise skip for non-innodb tables
if [ -f "$datadir/${olddb}/${tc}.ibd" ]; then
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
tc=$(echo $t | sed -e "s|-|@002d|g")
# only proceed if table's frm file is linked to innodb .ibd file
# otherwise skip for non-innodb tables
if [ -f "$datadir/${olddb}/${tc}.ibd" ]; then
echo "rm -f \"$datadir/${olddb}/${tc}.ibd\""
rm -f "$datadir/${olddb}/${tc}.ibd";
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
tc=$(echo $t | sed -e "s|-|@002d|g")
# only proceed if table's frm file is linked to innodb .ibd file
# otherwise skip for non-innodb tables
if [ -f "$datadir/${newdb}/${tc}.ibd" ]; then
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