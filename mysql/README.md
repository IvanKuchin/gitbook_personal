# MySQL

## Update MAX() + 1

Cледующая конструкция работать не будет, по причине рекурсивной ссылки на самого себя.

```sql
UPDATE `feed_images` SET `setID`=MAX(`setID`) + 1 WHERE `id`='xxx'
```

Можно использовать следующий трюк. Сделать таблицу из единственного значения:

```sql
 SELECT maxID FROM ( SELECT MAX(`setID`)+1 AS maxID FROM  `feed_images ) AS t
```

И затем использовать это значение для обновления:

```sql
UPDATE feed_images SET 
  setID = (SELECT maxID FROM (SELECT MAX(setID)+1 AS maxID FROM feed_images) AS t ) 
  WHERE tempSet='ххх'; 
```

## Long idle DB connection

Время жизни MySQL-connection по умолчанию 28800 сек = 8-часов.

```sql
show variables like "wait_timeout"; 
show variables like "interactive_timeout";
```

Applications that requires long idle DB connection (for example: chat-server), must periodically reset DB-connection.&#x20;

List of active DB connections:

```sql
SELECT * FROM  `PROCESSLIST`;
SHOW FULL PROCESSLIST;
```

## UTF8 vs UTF8mb4

There is not much difference in client application to work with UTF8 or UTF8mb4 DB. The only initialization phase must be changed

```
set names utf8mb4 instead of set names utf8
```

But big difference in DB. Working in utf8 character set DB is capable to store symbols from \ux0 to \uxFFFF (BMP Basic Multilingual Plane) wich is 5.88% of all possible Unicode code points (\ux0 to \ux10FFFF). UTF8mb4 capable to store whole UTF-8 available range (BMP + 0x10 extended planes or supplementary characters).&#x20;

To convert DB from utf8 to utf8mb4:

1. mysqldump
2. In table create statement replace: `CHARSET=utf8` to `CHARSET=utf8mb4 COLLATE = utf8mb4_unicode_ci`
3. Submit database back to mysql

## Log slow queries and queries w/o indexes

* Edit /etc/mysql/my.cnf

```
slow-query-log          = /var/log/mysql/mysql-slow.log
long_query_time = 2
log-queries-not-using-indexes
```

* Restart mysql service

{% hint style="warning" %}
Mysql might be protected by apparmor. Edit it config in /etc/apparmor.d/usr.sbin.mysqld



/proc/\*/status r,&#x20;

/sys/devices/system/node/ r,&#x20;

/sys/devices/system/node/node0/meminfo r,



Then restart it:  sudo systemctl restart apparmor.service
{% endhint %}

## Passwordless authentication

To avoid keep sensitive info (login/pass) in scripts, switch to login-path authentication. This will help to collaborate on development but keep password info out of public stores. Create login-path named local:

```
mysql_config_editor set –-login-path=local –-host=localhost –-user=root -p
```

Check that it has been created:

```
mysql_config_editor print –-login-path=local
```

Use mysql w/o login/pass in CLI:

```
mysql –-login-path=local -e “SELECT NOW();” www.connme.ru
```

## Seconds since epoch

Difference between:

```
SELECT TIMESTAMPDIFF(second, "1970-01-01", NOW())
```

and

```
SELECT UNIX_TIMESTAMP()
```

“Epoch” starts at&#x20;

* Jan 1, 1970 00:00:00 UTC
* Jan 1, 1970 03:00:00 MOW
* Dec 31, 1969 19:00:00 DST

![Difference between UNIX\_TIMESTAMP(date2) and TIMESTAMPDIFF(date1, date2)](../.gitbook/assets/epoch\_start.png)

| Language | Func                                              | Start time | Result |
| -------- | ------------------------------------------------- | ---------- | ------ |
| MySQL    | SELECT TIMESTAMPDIFF(second, "1970-01-01", NOW()) | Local TZ   |        |
| MySQL    | SELECT UNIX\_TIMESTAMP()                          | Epoch      | same   |
| bash     | date +%s                                          | Epoch      | same   |
| JS       | dNow = new Date(); dNow.getTime()                 | Epoch      | same   |
| C++      | time(NULL)                                        | Epoch      | same   |

## Select MAX() from table grouped by field

Example: Find ticket\_id-s with status "new". In the table below it would be the only ticket - 3&#x20;

| ticket\_id | timestamp | state            |
| ---------- | --------- | ---------------- |
| 1          |           | new              |
| 2          |           | new              |
| 3          |           | new              |
| 1          |           | customer pending |
| 1          |           | closed           |
| 2          |           | company pending  |



Helpdesk database designed the following way:&#x20;

Table: helpdesk\_tickets contains ticket.id and opener\_user.id&#x20;

Table: helpdesk\_history contains all updates with severity, status, updater\_user\_id and reference to ticket.id&#x20;

**Challenge**: SELECT all cases with latest status “new”.&#x20;

It is difficult due to helpdesk ticket doesn’t have latest status, updater\_user\_id, severity.&#x20;

* Walk through every single ticket and check latest state.&#x20;
  * Pros: easy to implement.&#x20;
  * Cons: very long process
* Keep latest state in helpdesk\_ticket table
  * Pros:
  * Cons: additional overhead due to synchronization two tables
  * Cons: non-scalable, if another field is required, table structure change required as well
* Real-time calculations in database (described below)
  * Pros: scalable

```sql
SELECT a.*, b.* FROM `helpdesk_ticket_history` a
	LEFT JOIN `helpdesk_ticket_history` b
    	ON a.`helpdesk_ticket_id`=b.`helpdesk_ticket_id
        AND a.`eventTimestamp`<b.`eventTimestamp
WHERE b.`helpdesk_ticket_id` IS NULL
```

Explanation of the above code:

Lines (1) - (3) take all permutations of ticket history by column `helpdesk_ticket_id`

Line (4) sorts permutations by time. Corner case: assume `table a` looks at most recent ticket history item, then `table b` will be NULL

Line (5) look for corner cases where `table b` is NULL, means that `table a` has most recent ticket history.



Next step is to create VIEW based on that query:

```sql
VIEW helpdesk_ticket_history_last_case_state_view ...
```



## SELECT latest response from helpdesk-engineer for all tickets

Create supplemental view containing only responses from helpdesk engineers

```sql
VIEW helpdesk_ticket_history_helpdesk_users_view

SELECT * FROM `helpdesk_ticket_history` 
         WHERE `user_id` IN (SELECT `id` FROM `users` WHERE `type`="helpdesk")

```

Repeat previous algorithm to helpdesk view

```sql
VIEW helpdesk_ticket_history_last_helpdesk_user_update_view

SELECT a.*, b.* FROM `helpdesk_ticket_history_helpdesk_users_view` a
	LEFT JOIN `helpdesk_ticket_history_helpdesk_users_view` b
    	ON a.`helpdesk_ticket_id`=b.`helpdesk_ticket_id
        AND a.`eventTimestamp`<b.`eventTimestamp
WHERE b.`helpdesk_ticket_id` IS NULL
```

Explanation of the above code word-to-word matches previous one.

## Why robot-helpdesk-user is not an option in current implementation?

**Challenge:** use dedicated robot-user for tasks like: case-assignment, case-close\_pending, case-closing. This way Customer will clearly negotiate that non-human has changed case status. Assign robot to helpdesk-role this makes him helpdesk-employee.

**Issues:**

* current implementation with `helpdesk_ticket_history_last_helpdesk_user_update_view` described [here ](./#select-latest-response-from-helpdesk-engineer-for-all-tickets)assumes that latest person from helpdesk is the case owner, then as soon as robot-user change the case history it become case-owner and next Customer-response will _NOT_ be visible to original human-helpdesk.
*   what if robot-user will not be a helpdesk role? Then the original human-helpdesk will be case owner after robot response.

    If robot is \_NOT\_ the helpdesk-er, then once robot closed the case, this action is not visible as helpdesk-action (again due to `helpdesk_ticket_history_last_helpdesk_user_update_view` is looking at helpdesk users only). As consequence all cases assigned to helpdesk-user won’t ever be visible as “closed”.

## How to copy custom fields from SoW\_X to SoW\_Y that are missed in SoW\_Y

Example: in the table below, the only field that must be copied from SoW 1 -> 2 is dayrate

| SoW ID | Custom Field Name | Custom Field Value |
| ------ | ----------------- | ------------------ |
| 1      | number            | 11.12              |
| 1      | dayrate           | 1000               |
| 2      | number            | 22                 |

Solution:

```sql
INSERT INTO `contract_sow_custom_fields` (   
	`contract_sow_id`,                        
	`var_name`,                               
	`title`,                                        
	`description`,                                  
	`type`,                                         
	`value`,
	`visible_by_subcontractor`,
	`editable_by_subcontractor`,
	`owner_user_id`,
    `eventTimestamp`
) 
SELECT 
	"5" AS `contract_sow_id`,              
	a.`var_name` AS `var_name`,
	a.`title` AS `title`,
	a.`description` AS `description`,
	a.`type` AS `type`,
	a.`value` AS `value`,
	a.`visible_by_subcontractor` AS `visible_by_subcontractor`,
	a.`editable_by_subcontractor` AS `editable_by_subcontractor`,
	a.`owner_user_id` AS `owner_user_id`,
	UNIX_TIMESTAMP() AS `eventTimestamp`   

FROM (SELECT * FROM `contract_sow_custom_fields` WHERE contract_sow_id="1") a
LEFT JOIN (SELECT * FROM `contract_sow_custom_fields` WHERE contract_sow_id="5") b
USING(`var_name`)
WHERE b.`id` is NULL
```

Explanation of code snippet above:

Lines (1) - (12) simple INSERT statement

Lines (13) - (23) SELECT statement that matches whatever must be inserted

Lines (25) - (27) build joined table from SoW1 and SoW5 by field `var_name`. As a result joined table will contains NULL in rows that are miss `var_name` matching between tables. If `var_name` values are missed in table a those will be NULL and vice versa about table b. &#x20;

Line (28) filter joined table by the rows that table b has NULL in.

## Master / Slave replication

![](../.gitbook/assets/MySQL\_replication.png)

Run two pods in Kubernetes stateful set: mysql-0 must be master, mysql-1 must be slave.&#x20;

Master config in /etc/mysql/mysql.conf.d/mysqld.cnf

```
server-id               	= 1
log_bin                 	= /var/log/mysql/mysql-bin.log
binlog_expire_logs_seconds   = 2592000
max_binlog_size   		= 100M
```

```bash
service mysql restart
```

```sql
mysql -u XXX -pxxxx 
CREATE USER 'replication'@'%' IDENTIFIED BY 'replication';
GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%';
GRANT FLUSH ON *.* TO 'replication'@'%';
```

Slave config in /etc/mysql/mysql.conf.d/mysqld.cnf

```
server-id = 2
```

```bash
service mysql restart
mysqldump -u replication -preplication --host=mysql-sts-0.mysql-svc --all-databases --master-data=2 > dump.sql
more dump.sql | grep MASTER
# -- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=5183;
mysql -u root -p < dump.sql
```

```sql
mysql -u root -p
CHANGE MASTER TO
MASTER_HOST='mysql-sts-0.mysql-svc',
MASTER_USER='replication',
MASTER_PASSWORD='replication',
MASTER_LOG_FILE='mysql-bin.000002',
MASTER_LOG_POS=5183;
start slave;
show slave status;
```

## NULL values

Use case for `NULL` values: If we need benefits of `InnoDB` ie. constraint checks and UI reference between tables, but sometime tables might not have value in that field, then `NULL` is the way to solve the issue.&#x20;

On the code side `NULL` will cause all sorts of issues. &#x20;

* This is another corner case to check.
* It is not future-proof
* Requires NULL-specific type (for example: `StringNULL` instead of just `String`)

To address this problem use `COALESCE()` function in SQL-query.

```sql
	SELECT
		name,
		COALESCE(other_field, '') as otherField
	WHERE id = "123"
```

Line (3) - `COALESCE` returns first non-NULL value
