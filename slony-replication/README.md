This document will explain how to set up Slony replication with Postgres.

**** 1
- build Master and Slave Postgres servers
- build Replica servers you want to replicate tables to from the Postgres server

**** 2
- run config management if needed
- apt-get install postgresql-9.3 slony1-2-bin postgresql-9.3-slony1-2 on postgres servers
- apt-get install postgresql-9.3-slony1-2 on replicas
- run config management again
- disable any cronjob

**** 3
on all servers:
- update pg_hba.conf to allow connections
- update postgresql.conf to listen on *
- restart postgres daemon
- createuser -s mainuser slon_local slon_remote replication repl
- createdb -O mainuser maindb
- createlang plpgsql maindb
- echo "ALTER USER mainuser WITH PASSWORD 'abc123'" | psql

**** 4
- on standy node follow these steps: https://wiki.postgresql.org/wiki/Binary_Replication_Tutorial#Streaming_Replication
- save recovery.conf to /root
standby_mode = on
primary_conninfo = 'host=postgres1.domain.com user=repl password=abc123'
trigger_file = '/tmp/postgresql-repl.trigger'
- on master run "SELECT pg_start_backup('backup');"
- rsync -arv /var/lib/postgresql/9.3/main root@postgres2.domain.com:/var/lib/postgresql/9.3/
- on master run "SELECT pg_stop_backup();"

**** 4
- import maindb schema to replicas
- import maindb public to postgres1

**** 5
- on master create /etc/slony1/clustername/slon.conf
<<<
syslog=1
log_timestamp=1
log_level=0
cluster_name='clustername'
conn_info="dbname=maindb host=1.2.3.4 user=slon_remote password=abc123"
>>>
- create /etc/slony1/slon_tools.conf
<<<
$CLUSTER_NAME = 'clustername';
$PIDFILE_DIR = '/var/run/slony1';
$LOGDIR = '/var/log/slony1';
$MASTERNODE = 1;

add_node(host => 'postgres1.domain.com', dbname => 'maindb', port =>5432,
        user=>'slon_local', password=>'abc123', node=>1 );
add_node(host => 'replica0.domain.com', dbname => 'maindb', port => 5432, 
        user => 'slon_remote', password=>'abc123', node=>2 );

$SLONY_SETS = {
 "tables" => {
        "set_id" => 1,
        "pkeyedtables" => [
                "public.table1",
                "public.table2",
        ],
        "sequences" => [
                "public.table2_id_seq",
                "public.table1_id_seq",
        ],
 },
};


1;
>>>
- run slony scripts to set up replication (make sure there are no duplicate keys in the public.table1 and public.table2, otherwise drop and import again)
- on master edit /etc/default/slony1 and add all nodes
- restart slony

**** 6
- to upgrade Postgres 9.3 to 9.6 in place, see this guide: https://github.com/dvuk84/docs/blob/master/upgrade-postgres-inplace.md

**** 7
- pg_dump takes 2h
- importing takes roughly the same time
- upgrading takes 1-1.5h
- analyze takes 20-30min
- full vacuum takes 2-3h
- vacuum takes about 10min

**** 8
Every time an UPDATE or DELETE statement is executed, Postgres will update the record by creating a new copy of the row and leaving the 
old one behind as a dead row or dead tuple. So when a transaction is to be commited, if it aborts before it finished then then old row should be restored and the new
one removed and if it's successfull then the new one will be used and and old one will be removed. That's what MVCC type database engine is. Vacuum removes these dead rows and releases space for new ones.
