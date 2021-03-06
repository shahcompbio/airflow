# airflow

[Documentation from forked repo](https://github.com/puckel/docker-airflow/blob/master/README.md). Refer to this documentation when making changes to the production deployment.

## Shah lab Production Deployment
This deployment employs the Celery scheduler. This scheduler adds tasks to a queue using Redis, and remote workers pick them off of the queue and execute them. The workers run on separate processes from the scheduler and are therefore fully isolated makeing the system highly scalable — heavy and expanding loads can be managed by simply adding additional workers and instantly increasing the throughput of the system.

This deployment employs role based access control. 

1. Pull Airflow.

```bash
    git clone https://github.com/shahcompbio/airflow.git
    cd airflow
```

2. Start with a clean slate. Verify there are no containers, images or volumes.

```bash
    docker system prune -a -f
    docker images 
    docker ps -a  
    docker volume ls 
```

3. Add ssl certificates - copy public.key and private.key into config dir.

```bash
    cp [PUBLIC_KEY_SOURCE_PATH] config/public.key
    cp [PRIVATE_KEY_SOURCE_PATH] config/private.key
```

4. Create database dir.

```bash
    mkdir pgdata
```
    
5. Copy credentials template file, add passwords to .envs/.docker-compose-CeleryExecutor.sh and source the file.
    
```bash
    cp .envs/.docker-compose-CeleryExecutor.sh.template .envs/.docker-compose-CeleryExecutor.sh
    source .envs/.docker-compose-CeleryExecutor.sh
```
    
6. Verify - there should be no WARNINGS and all environment variables in docker-compose file should be substituted.
 
 ```bash
    docker-compose -f docker-compose-CeleryExecutor.yml config  
 ```
 
7. Start server.
 
 ```bash
    docker-compose -f docker-compose-CeleryExecutor.yml up -d
 ```

8. Create an admin user.

  ```bash
    docker-compose -f docker-compose-CeleryExecutor.yml exec webserver /entrypoint.sh bash
    airflow create_user --help
    airflow create_user -r Admin -u admin -e admin@example.com -f admin -l user -p test
   ```
   
 9. Log into UI at https://DEPLOYMENT_URL[:PORT].

## Adding DAGS

1. User: Add DAG script to the dags dir.

2. User: Add script dependencies to requirements.txt.

3. User: Push to repo 

4. Admin: Pull into prod environment. 

5. Admin: Restart server to install dependencies.

 ```bash
    cd [AIRFLOW_DIR]
    docker-compose -f docker-compose-CeleryExecutor.yml down
    docker-compose -f docker-compose-CeleryExecutor.yml up -d
 ```

6. Admin: Test DAG locally. 

```bash
    docker-compose -f docker-compose-CeleryExecutor.yml exec webserver /entrypoint.sh bash
    airflow test --help
    airflow test tutorial print_date 2015-06-01
   ```

7. Airflow automatically picks up the new DAG and displays it on the UI.
