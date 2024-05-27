**DevOps Assessment Task – Python Flask**

1. As First step launched an EC2 instance from AWS, then installed Docker and Docker Compose on the server. After that downloaded the Application files provided and stored in local and pushed to Github repo then cloned to the EC2 server
2. Created a **project** directory and put the application files inside the directory along with the Dockerfile and docker-compose.yml file
```
.
└── project
    ├── Dockerfile
    ├── docker-compose.yml
    └── python_flask
        ├── README.md
        ├── app.py
        ├── create_db.py
        ├── drop_db.py
        └── requirements.txt
```
**Dockerfile:**
```
FROM python:3.10                                               # base image
WORKDIR /app                                                   # set work directory inside container
COPY /python_flask /app                                        # copy all app files inside container
RUN pip install --no-cache-dir --upgrade pip                   # upgrade pip 
RUN pip install --no-cache-dir -r requirements.txt             # install all required python dependencies
EXPOSE 5000                                                    # expose port on container
ENTRYPOINT ["python","app.py"]                                 # execute these commands when container starts
```
let's break down this Dockerfile step by step:

1. **FROM python:3.10 :** This line specifies the base image for Dockerfile, here we have taken python:3.10 from Docker Hub as base image.
2. **WORKDIR /app :** This command will help to sets the the Working directory as **/app** inside the container.When we exec into the container it will be in the same path.
3. **COPY /python_flask /app :** This command is for to copy all contents of local directory **python_flask** to the **/app** directory insude the container.
4. **RUN pip install --no-cache-dir --upgrade pip :** This command upgrades pip,the Python package installer,to the latest version and --no-cache-dir flag prevents caching of downloaded packages.
5. **RUN pip install --no-cache-dir -r requirements.txt :** This will help to install all python dependencies listed in **requirements.txt**
6. **EXPOSE 5000 :** This exposes port 5000 on the Docker container.
7. **ENTRYPOINT ["python","app.py"] :** This specifies the command that will be executed when the Docker container starts.

**docker-compose.yml :**
```

version: '3.8'                                                              # docker-compose version
services:                                                                   # defines all container the will make up the multi container application
  app:
    build: .                                                                # build the app container from the Dockerfile we defined
    environment:                                                            # set all required env variables
      - APP_HOST=0.0.0.0
      - REDIS_HOST=127.0.0.1
      - REDIS_PORT=6379
      - DATABASE_URI=mysql+pymysql://<user>:<password>@<host>:3306/<db_name>
    ports:                                                                  # maps host port to container port
      - "5000:5000"
    depends_on:                                                             # app service depends on other 2 service
      - redis
      - db
    healthcheck:                                                            # configure the healthcheck of service
      test: ["CMD", "curl", "-f", "http://localhost:5000/status"]
      interval: 30s
      timeout: 3s
      retries: 3
  redis:                                                                    # create redis container
    image: "redis:latest"
  db:
    image: "mysql:latest"                                                   # create mysql container
    environment:                                                            #  sets env varible for mysql
      - MYSQL_DATABASE=db_name
      - MYSQL_USER=username
      - MYSQL_PASSWORD=password
      - MYSQL_ROOT_PASSWORD=root_password
```
let's break down this docker-compose.yml step by step:

1. **version: '3.8' :** This indicates the version of docker-compose being used, here it is 3.8
2. **services :** This section defines the container service.

    **app:** This section defines the application container  It's built from the current directory (build: .) from the Dockerfile defined. It sets environment variables 
             such as APP_HOST, REDIS_HOST,REDIS_PORT, and DATABASE_URI. It exposes port 5000 on the host machine and maps it to port 5000 in the container (ports: 
             "5000:5000"). 
             It depends on the redis and db services. It also defines a health check to verify the health of the application using a curl command every 30 seconds.

    **redis:** In this section the redis container will be created with latest image.

    **db:** In this section db container will be created with image tag latest also it sets environment variables for the MySQL database such as MYSQL_DATABASE, MYSQL_USER, 
            MYSQL_PASSWORD, and MYSQL_ROOT_PASSWORD.

Next step is to build and start the application with 
   **docker-compose up --build -d**

checked container statsu by **docker ps**

```
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS                        PORTS                                       NAMES
71d8a43a59bc   project-app    "python app.py"          About a minute ago   Up About a minute (healthy)   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   project-app-1
9d7800ab6035   mysql:latest   "docker-entrypoint.s…"   About a minute ago   Up About a minute             3306/tcp, 33060/tcp                         project-db-1
a34748888e05   redis:latest   "docker-entrypoint.s…"   About a minute ago   Up About a minute             6379/tcp                                    project-redis-1
```
Checked connectivity from project-app-1 to other 2 containers through ping command and it showing connectivity
```
[root@ip-172-31-22-182 project]# docker exec -it project-app-1 /bin/bash
root@71d8a43a59bc:/app# ping project-db-1
PING project-db-1 (172.18.0.3) 56(84) bytes of data.
64 bytes from project-db-1.project_default (172.18.0.3): icmp_seq=1 ttl=127 time=0.099 ms
64 bytes from project-db-1.project_default (172.18.0.3): icmp_seq=2 ttl=127 time=0.052 ms
64 bytes from project-db-1.project_default (172.18.0.3): icmp_seq=3 ttl=127 time=0.065 ms
64 bytes from project-db-1.project_default (172.18.0.3): icmp_seq=4 ttl=127 time=0.065 ms

--- project-db-1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3113ms
rtt min/avg/max/mdev = 0.052/0.070/0.099/0.017 ms
root@71d8a43a59bc:/app# ping project-redis-1
PING project-redis-1 (172.18.0.2) 56(84) bytes of data.
64 bytes from project-redis-1.project_default (172.18.0.2): icmp_seq=1 ttl=127 time=0.094 ms
64 bytes from project-redis-1.project_default (172.18.0.2): icmp_seq=2 ttl=127 time=0.070 ms
64 bytes from project-redis-1.project_default (172.18.0.2): icmp_seq=3 ttl=127 time=0.066 ms

--- project-redis-1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2110ms
rtt min/avg/max/mdev = 0.066/0.076/0.094/0.012 ms

```

checked the databse table in db container and verified
```
[root@ip-172-31-22-182 project]# docker exec -it project-db-1 /bin/bash
bash-5.1# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.4.0 MySQL Community Server - GPL

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| db_name            |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```




 

