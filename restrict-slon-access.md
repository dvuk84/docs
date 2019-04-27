When doing Slony replication, **slon_remote** will normally be created with Superuser privileges. This document will explain how to
restrict access to **slon_remote** in production environment with no downtime. You will need to update \<DATABASE\>, \<SCHEMA\> and
\<TABLE1\> with relevant details.

```
# MASTER

---- PRE-CHANGE
* ssh postgres.domain.com
* sudo -i
* su - postgres
* psql
* \du+
* \c <DATABASE>
* \z public.<TABLE1>
* \z _<SCHEMA>.*
* SELECT table_catalog, table_schema, table_name, privilege_type FROM information_schema.table_privileges WHERE grantee = 'slon_remote';

---- APPLY CHANGES
* ALTER USER slon_remote WITH NOSUPERUSER NOCREATEROLE NOCREATEDB;
* GRANT USAGE ON SCHEMA _<SCHEMA> TO slon_remote;
* GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA _<SCHEMA> TO slon_remote;
* GRANT SELECT ON public.<TABLE1> TO slon_remote;
* GRANT SELECT ON ALL TABLES IN SCHEMA _<SCHEMA> TO slon_remote;
* GRANT ALL ON _<SCHEMA>.sl_nodelock TO slon_remote;

---- POST-CHANGE
* \du+
* \c <DATABASE>
* \z public.soa
* \z _<SCHEMA>.*
* SELECT table_catalog, table_schema, table_name, privilege_type FROM information_schema.table_privileges WHERE grantee = 'slon_remote';
```
