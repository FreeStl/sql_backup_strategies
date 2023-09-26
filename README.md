Comparison of MySql backup strategies

# Full Backup
- This approach is simple, but not flexible, consumes a lot of memory and doesn't allow to rollback to specific point of time.
1) start mysql using docker-compose.yml. Path to backup should be backups/full
2) Create table and insert 10k rows. Use create and insert sql scripts in commands folder.
3) Backup data using command 
```shell
mysqldump -u root -ppassword my_db books --flush-logs > ./backup/full.sql
```
It will save data
4) Remove mysql volume to clear data. Then restart pod.
5) Restore data with command:
```shell
mysql -u root -ppassword my_db < /backup/full.sql
```

#  Differential Backup
- Each backup is based on last full backup. This approach is space efficient and fast. But, if you lose last full backup, you break the chain and lose opportunity to rollback to some backups 
1) start mysql using docker-compose.yml. Path to backup should be backups/full
2) Create table and insert 10k rows. Use create and insert1 sql scripts in commands folder.
3) Make full backup
 ```shell
mysqldump -u root -ppassword my_db students --flush-logs > /backup/full.sql
```
4) insert additional data. Run insert2 and insert3 scripts
5) Make diff backup, bqsed on full backup
```shell
mysqlbinlog /backup/log-bin.000001 /backup/log-bin.000002 > /backup/diff-backup.sql
```
5) Remove mysql volume to clear data. Then restart pod.
6) Restore db to first full backup
```shell
mysql -u root -ppassword my_db < /backup/full.sql
```
7) Restore rest data using differential backup
```shell
mysql -u root -ppassword my_db < /backup/diff-backup.sql
```
All data restored

# Incremental backup
- Similar concept as  Differential Backup. However, it is faster, less memory consuming and less reliable. After you do first full backup,
each next backup is based on previous differential backup. This approach is space efficient and fast. But, if you lose any backup, 
you break the chain and lose opportunity to rollback to prev backups

# Reverse Delta Backup
- Reverse of Differential backup: latest backup is alvays full backup. This approach provides as with faster rollback speed.
- First time we need to make backup, we just create full backup
- Next time we need to make backup:
  1) We create new full backup
  2) We find and save a difference between previous backup and latest backup
  3) Delete previous full backup
  4) Therefore, we have only one full latest backup. If we want to restore earlier data, we forst rollback to latest full backup, 
      then apply delta

# CDP
All changes in bit format are continuously stored on another server. It gives opportunity to restore db to exact moment, 
or rollback exact number of updates. This is the safest approach. However, it is very memory and cpu consuming


