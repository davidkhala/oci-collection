## Current limitations
- Inbound Replication is not managed functionality. The user is reponsible for configuring and maintaining the channel
- Only Row-based replication supported. Only GTID-based replication is supported.
- Multi-source replication is not supported. Replication filters are not supported.
- Changes to the mysql schema are not replicated and cause replication to stop.
- Source must run with the same `lower_case_table_names` value as target DB System. 
- The inbound applier (Target side) runs under the privileges of the DB System's admin user. 

### Source side Configuration
- A replication user must be configured on the source server, for example a user named `rpluser001` with the password `Rpl001#!`
     ```
     CREATE USER rpluser001@'%' IDENTIFIED BY 'Rpl001#!' REQUIRE SSL; ## 'REQUIRE SSL' requires the server to permit only encrypted connections
     GRANT REPLICATION SLAVE on *.* to rpluser001@'%';
     ```
- The MySQL source server's `GTID_MODE` variable must be set to `ON`. 
    - Verify the current `GTID_MODE` by `select @@gtid_mode;`
        - MDS HA has `GTID_MODE` default to `ON` 
    - If `GTID_MODE` is off, 
        - Find more in https://dev.mysql.com/doc/refman/8.0/en/replication-mode-change-online-enable-gtids.html
        - TODO try this instead 'https://lefred.be/content/using-mysql-database-service-for-wordpress/'

### Target side Configuration
- Create a replication channel on OCI Console
- `It is not possible to create a channel on a DB System with High Availability enabled.`
   - MDS HA as Source, Heatwave as Target. :heavy_check_mark: Supported
   - Heatwave as Source, MDS HA as Target. :x: Not supported

## Trouble shoot
- Q: after creating replication channel status coming as `need attention`. Error message discplayed as " The Channel is not receiving transactions due to errors (MY-2003, Fri Jul 02 10:23:48 GMT 2021). The connection may be in progress"
    - Confirm both in same VCN and properly configure security list. 
    - login into MDS and run sql to view error stack: `show replica status;`
- Q: Got fatal error 1236 from master when reading data from binary log: 'Cannot replicate because the master purged required binary logs. Replicate the missing transactions from elsewhere, or provision a new slave from backup. Consider increasing the master's binary log expiration period. The GTID set sent by the slave is ''

- `select * from performance_schema.replication_connection_status` 
    - shows the current status of the I/O thread that handles the replica's connection to the source
    - run on Source
- `select * from performance_schema.replication_applier_status_by_worker`
    - shows details of the transactions handled by applier threads on a replica
    - run on Target
- Other [TroubleShoot](https://docs.oracle.com/en-us/iaas/mysql-database/doc/troubleshooting.html#MYAAS-GUID-A681C832-E480-41BD-B51F-5818045C35C5)
### Table formatter for `show replica status`
| Replica_IO_State                 | Source_Host  | Source_User | Source_Port | Connect_Retry | Source_Log_File   | Read_Source_Log_Pos | Relay_Log_File                       | Relay_Log_Pos | Relay_Source_Log_File | Replica_IO_Running | Replica_SQL_Running | Replicate_Do_DB | Replicate_Ignore_DB | Replicate_Do_Table | Replicate_Ignore_Table | Replicate_Wild_Do_Table | Replicate_Wild_Ignore_Table | Last_Errno | Last_Error | Skip_Counter | Exec_Source_Log_Pos | Relay_Log_Space | Until_Condition | Until_Log_File | Until_Log_Pos | Source_SSL_Allowed | Source_SSL_CA_File | Source_SSL_CA_Path | Source_SSL_Cert | Source_SSL_Cipher | Source_SSL_Key | Seconds_Behind_Source | Source_SSL_Verify_Server_Cert | Last_IO_Errno | Last_IO_Error | Last_SQL_Errno | Last_SQL_Error | Replicate_Ignore_Server_Ids | Source_Server_Id | Source_UUID                          | Source_Info_File        | SQL_Delay | SQL_Remaining_Delay | Replica_SQL_Running_State                                | Source_Retry_Count | Source_Bind | Last_IO_Error_Timestamp | Last_SQL_Error_Timestamp | Source_SSL_Crl | Source_SSL_Crlpath | Retrieved_Gtid_Set                       | Executed_Gtid_Set                        | Auto_Position | Replicate_Rewrite_DB | Channel_Name        | Source_TLS_Version | Source_public_key_path | Get_Source_public_key | Network_Namespace |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | 
| Waiting for source to send event | 192.168.1.73 | davidkhala  |        3306 |            60 | binary-log.000012 |                 196 | relay-log-replication_channel.000018 |           413 | binary-log.000012     | Yes                | Yes                 |                 |                     |                    |                        |                         |                             |          0 |            |            0 |                 196 |             724 | None            |                |             0 | No                 |                    |                    |                 |                   |                |                     0 | No                            |             0 |               |              0 |                |                             |        685745206 | 581ef611-0ee0-11ec-8948-02001700aaaa | mysql.slave_master_info |         0 |                NULL | Replica has read all relay log; waiting for more updates |                  0 |             |                         |                          |                |                    | afabb0fd-99c7-4fb9-ba44-0419b16b3a20:1-2 | afabb0fd-99c7-4fb9-ba44-0419b16b3a20:1-2 |             1 |                      | replication_channel | TLSv1.2,TLSv1.3    |                        |                     1 | mysql             |

