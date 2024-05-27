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
FROM python:3.10
WORKDIR /app
COPY /python_flask /app
RUN pip install --no-cache-dir --upgrade pip
RUN pip install --no-cache-dir -r requirements.txt
EXPOSE 5000
ENTRYPOINT ["python","app.py"]
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

version: '3.8'
services:
  app:
    build: .
    environment:
      - APP_HOST=0.0.0.0
      - REDIS_HOST=127.0.0.1
      - REDIS_PORT=6379
      - DATABASE_URI=mysql+pymysql://<user>:<password>@<host>:3306/<db_name>
    ports:
      - "5000:5000"
    depends_on:
      - redis
      - db
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/status"]
      interval: 30s
      timeout: 3s
      retries: 3
  redis:
    image: "redis:latest"
  db:
    image: "mysql:latest"
    environment:
      - MYSQL_DATABASE=db_name
      - MYSQL_USER=username
      - MYSQL_PASSWORD=password
      - MYSQL_ROOT_PASSWORD=root_password
```
let's break down this docker-compose.yml step by step:

1. **version: '3.8' :** This indicates the version of docker-compose being used, here it is 3.8
2. **services :** This section defines the container service.

    **app:** This section defines the application container  It's built from the current directory (build: .) from the Dockerfile defined. It sets environment variables such 
             as APP_HOST, REDIS_HOST,REDIS_PORT, and DATABASE_URI. It exposes port 5000 on the host machine and maps it to port 5000 in the container (ports: - "5000:5000"). 
             It depends on the redis and db services. It also defines a health check to verify the health of the application using a curl command every 30 seconds.

    **redis:** In this section the redis container will be created with latest image.

    **db:** In this section db container will be created with image tag latest also it sets environment variables for the MySQL database such as MYSQL_DATABASE, MYSQL_USER, 
            MYSQL_PASSWORD, and MYSQL_ROOT_PASSWORD.

Next step is to build and start the application with 
   **docker-compose up --build -d**

 

