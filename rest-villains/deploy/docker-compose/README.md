Files in here are generated during build and will be over-written if modified directly.

See [`quarkus-super-heroes/.github/workflows/create-deploy-resources.yml`](../../../.github/workflows/create-deploy-resources.yml) and [`quarkus-super-heroes/scripts/generate-docker-compose-resources.sh`](../../../scripts/generate-docker-compose-resources.sh).

if you failed when running container becase error while it starting postgres like this ```ls: cannot access '/docker-entrypoint-initdb.d/': Operation not permitted```
you can do like these ways : https://www.onooks.com/docker-compose-postgres-docker-entrypoint-initdb-d-init-sql-permission-denied/

1. disable SELinux

    ```console 
   $ echo SELINUX=disabled > /etc/selinux/config
   $ SELINUXTYPE=targeted >> /etc/selinux/config
   $ setenforce 0
   ```
2. use “build” way

   You should use “build” way to create postgres service
   And DO NOT setting the volume for init.sql, it will cause the permission problem.
    ```
    postgres:
        build: ./postgres
   ```
   Create a Dockerfile for postgres like this
   ```
    FROM postgres:12
    COPY ./init.sql /docker-entrypoint-initdb.d/init.sql
    CMD ["docker-entrypoint.sh", "postgres"]
    ```
   Then it should works out. Hope my answer would help you

    I had a similar problem when using the ADD command.
    
    When using ADD (eg to download a file) the default chmod value is 711. When using the COPY command the chmod will match the hosts chmod of the file where you copy it from. The solution is to set the permission before copying, or change them in your Dockerfile after they’ve been copied.
    
    It looks like there will finally be a “COPY –chmod 775” flag available in the upcoming docker 20 which will make this easier.
    
    https://github.com/moby/moby/issues/34819
    
    When the sql file in /docker-entrypoint-initdb.d/ has a permission of 775 then the file is run correctly.
    
    You can test this within the image (override entry point to /bin/bash) using: docker-entrypoint.sh postures
