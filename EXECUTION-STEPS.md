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

