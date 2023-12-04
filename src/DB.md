# DB

Owner: Mojtaba Safaei
Verification: Verified

- postgresql
    
    change user to postgres:
    
    sudo su - postgres
    
    run dump command:
    
    pg_dumpall -c -U postgres > dump_`date +%d-%m-%Y""%H%M_%S`.sql
    
    compress dump file:
    
    pigz -9 -p 4 filename
    
- Postgres Dump
    
    change user to postgres:
    
    sudo su - postgres
    
    run dump command:
    
    pg_dumpall -c -U postgres > dump_`date +%d-%m-%Y""%H%M_%S`.sql
    
    compress dump file:
    
    pigz -9 -p 4 filename
    
- Mysql Dump
    1. Login into the pod   `kubectl exec -it <pod> -n <namespace> -- /bin/bash`
    2. run mysqldump from within the pod and use `tmp` to write the file `mysqldump -u root -p <db_name> >  /tmp/file-`date +%d-%m-%Y"-"%H%M_%S`.sql`
    3. Copy the file from the pod   `kubectl cp <namespace>/<pod>:/tmp/file.sql file.sql`