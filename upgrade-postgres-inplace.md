This document will show how to upgrade Postgres 9.3 to 9.6. It's an in-place upgrade.
```
# MASTER FIRST, THEN STANDBY

---- PRE-UPGRADE
* make sure there is enough space in /var for a second copy of the database
* make sure to take a database dump in case we need to restore, this will take about 4h
* stop Apache on all web servers to bring up the holding page
* stop queue subscribers on the web servers
* disable puppet on the web servers
* disable puppet on postgres servers
* stop backup cronjobs on postgres servers

---- UPGRADE
* stop postgresql on both database servers
* stop slony cluster on master
* modify /etc/postgresql/9.3/main/postgresql.conf and change port number to 5433
* disable 9.3 as installing 9.6 will attempt to start the daemon
* install postgresql-9.6 postgresql-9.6-slony1-2
* stop postgresql again
* copy /etc/postgresql/9.3/main/postgresql.conf to /etc/postgresql/9.6/main/postgresql.conf
* copy /etc/postgresql/9.3/main/pg_hba.conf to etc/postgresql/9.6/main/pg_hba.conf
* modify /etc/postgresql/9.6/main/postgresql.conf and comment out checkpoint_segments and add min_wal_size = 5 and max_wal_size = 10 and also make sure to update any references to 9.3 !!!
* link both config files as pg_upgrade will be looking in /var and not /etc
  ln -s /etc/postgresql/9.3/main/postgresql.conf /var/lib/postgresql/9.3/main/
  ln -s /etc/postgresql/9.6/main/postgresql.conf /var/lib/postgresql/9.6/main/
  ln -s /etc/postgresql/9.3/main/conf.d/ /var/lib/postgresql/9.3/main/
  ln -s /etc/postgresql/9.6/main/conf.d/ /var/lib/postgresql/9.6/main/
* make sure postgres isn't running
* become postgres 
* run:
  /usr/lib/postgresql/9.6/bin/pg_upgrade --check --old-datadir=/var/lib/postgresql/9.3/main --new-datadir=/var/lib/postgresql/9.6/main --old-bindir=/usr/lib/postgresql/9.3/bin --new-bindir=/usr/lib/postgresql/9.6/bin
* if no errors, proceed to upgrade without --check
* disable old cluster:
  mv /etc/postgresql/9.3/main/postgresql.conf /etc/postgresql/9.3/main/postgresql.conf.DISABLED
* copy recovery.conf on standy from 9.3 to 9.6 dir
* start the new server (start master first, then standby) and verify version with psql -c "SELECT version();"
* create a test database and see if replication works, if it doesn't and the databases are different, stop standby and rsync the files across
  rsync -avr postgres1.domain.com:/var/lib/postgresql/9.6/main/ /var/lib/postgresql/9.6/main
* run vacuum and analyze on lcn database
  /usr/lib/postgresql/9.6/bin/vacuumdb --dbname lcn --analyze-in-stages

---- POST-UPGRADE
* start Slony cluster
* test replication
* remove maintenance page
* run puppet to pull the postgres module
* enable cronjobs
* remove 9.3 packages and the old database files, otherwise there won't be enough space on the server:
  apt-get remove postgresql-9.3 postgresql-9.3-slony1-2 postgresql-client-9.3
  dpkg --purge postgresql-9.3
  rm -rf /var/lib/postgresql/9.3
  ```
