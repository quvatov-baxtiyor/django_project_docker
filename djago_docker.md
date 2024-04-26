Writing dockerfile for django project
1. create folder for your new project ``django-docker-db``
2. open this folder from your terminal(cmd)
3. run this command
    ```
    python3 -m venv venv
   ```
   for linux,mac:
   ```
    source venv/bin/activate
   ```
   for windows:
   ```
    source venv/Scripts/activate
   ```
   ```
    pip3 install django psycopg2-binary
    pip3 freeze > requirements.txt
    django-admin startproject config .
   ```
4. Change settings.py ``databases`` part. file ``config/settings.py``. 
    old version:
   ```
    DATABASES = {
        'default':
           {
             'ENGINE': 'django.db.backends.sqlite3',
             'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
           }
    }
   ```
   new version:
   ```
      DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql',
            'NAME': os.getenv("DATABASE_NAME"),
            'USER': os.getenv("DATABASE_USER"),
            'PASSWORD': os.getenv("DATABASE_PASSWORD"),
            'HOST': os.getenv("DATABASE_HOST"),  # set in Docker run
            'PORT': os.getenv("DATABASE_PORT"),
        }
    }

   ```

5. Let`s start writing ``Dockerfile-django``
   ```
   FROM python:3.11-slim-buster

   WORKDIR /app
   
   COPY . .
   
   RUN pip3 install -r requirements.txt
   
   EXPOSE 8000
   
   CMD ["python3","manage.py","runserver","0.0.0.0:8000"]
   ```
6. We should create 2 container. So they should connect each other. For connection, we should create new network. Creating network run this command:
    `` docker network create {my_netowrk}``
7. Let`s begin with database container. For running database container run this command:
   ```
    docker run -d  --name django_db_postgres  -p 5434:5432 \
    --network {my_network} \
    -e POSTGRES_PASSWORD=mypassword \
    -e POSTGRES_USER=testuser \
    -e POSTGRES_DB=testdb \
    postgres:16
   ```
8. So, now we should create docker image and container from Dockerfile. 
   creating image:
   ```
      docker build -f Dockerfile-django -t {image-name}:v1 .
   ```
9. Now make container for django, for creating this container run this command:
   ```
    docker run -d --name mydjangoapp_container \
      --network {my_network} \
      -e "DATABASE_HOST=django_db_postgres" \
      -e "DATABASE_PORT=5432" \
      -e "DATABASE_USER=testuser" \
      -e "DATABASE_PASSWORD=mypassword" \
      -e "DATABASE_NAME=testdb" \
      -p {port_number}:8000 \
      {image-name}
   ```



10. If you open browser and enter ``http://localhost:{port_number}``, you can see rocket
11. run this command ``docker ps``. You can see all running containers in your computer. It looks like that:![img.png](img.png)
12. Get your running django container ID and enter this container. for this `` docker exec -it {container_id} /bin/bash``
13. ![img_2.png](img_2.png)
   run `` python3 manage.py migrate``. If you see all OKs, everything is working. otherwise you have some mistakes.