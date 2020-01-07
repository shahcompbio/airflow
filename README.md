# airflow

## [Original Documentation](https://github.com/puckel/docker-airflow/blob/master/README.md)

## Shah lab Production Deployment
This deployment employes the Celery scheduler. This scheduler adds tasks to a queue using Redis, and remote workers pick them off of the queue and execute them. The workers run on separate processes from the scheduler and are therefore fully isolated makeing the system highly scalable — heavy and expanding loads can be managed by simply adding additional workers and instantly increasing the throughput of the system.

This deployment employs role based access control. 

1. Start with a clean slate. Verify there are no containers, images or volumes

```bash
    docker system prune -a -f
    docker images 
    docker ps -a  
    docker volume ls 
```

2. Add ssl certificates - copy public.key and private.key into config dir

```bash
    cp [PUBLIC_KEY_SOURCE_PATH] config/public.key
    cp [PRIVATE_KEY_SOURCE_PATH] config/private.key
```

3. Create database dir

```bash
    mkdir pgdata
```
    
4. Copy credentials template file, add passwords to .envs/.docker-compose-CeleryExecutor.sh and source the file
    
```bash
    cp .envs/.docker-compose-CeleryExecutor.sh.template .envs/.docker-compose-CeleryExecutor.sh
    source .envs/.docker-compose-CeleryExecutor.sh
```
    
 5. Verify - there should be no WARNINGS and all environment variables in docker-compose file should be substituted.
 
 ```bash
    docker-compose -f docker-compose-CeleryExecutor.yml config  
 ```
 
 6. Start server
 
 ```bash
    docker-compose -f docker-compose-CeleryExecutor.yml up -d
 ```

  7. Create an admin user

  ```bash
    docker-compose -f docker-compose-CeleryExecutor.yml exec webserver /entrypoint.sh bash
    airflow create_user --help
    airflow create_user -r Admin -u admin -e admin@example.com -f admin -l user -p test
   ```
   
  8. View UI at https://DEPLOYMENT_URL:8080

## Adding DAGS

1. Add DAG script to the dags dir.

2. Add script dependencies to requirements.txt.

3. Push to repo and pull into prod environment. 

4. Restart server to install dependencies

 ```bash
    docker-compose -f docker-compose-CeleryExecutor.yml down
    docker-compose -f docker-compose-CeleryExecutor.yml up -d
 ```

5. Airflow automatically picks up the new DAG and displays it on UI.
