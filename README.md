# Scenario 1

Use one heading for each problem.
Write what was wrong and how you fixed it.
Paste the config you changed (only the changed part).
Paste the commands you used.
Write every step you tried, even guesses.

English is better. Persian is OK.

## Problem 1: (short name)

the vm can not connect to internet because of dns . 

How I fixed it:

i change dns from 127.0.0.1 to 8.8.8.8 with vim at first beacuse is link to systemd i can not overwrite it .so delete it .then create new one.

code i used:
    rm /etc/resolv.conf
    echo "nameserver 8.8.8.8" > /etc/resolv.conf
 then install docker compose with:
     apt update && apt install -y docker-compose
         docker-compose up -d
then check with curl:    curl -i http://localhost/graph





## Problem 2: get 502.
means nginx is working but the backend give nothing.
In a Docker Compose setup, a 502 usually means:

The Backend is still starting up: It might be running migrations or waiting for the Database to be ready.
The Backend crashed: It started, but failed due to a configuration error.
Network/Port Mismatch: Nginx is looking for the backend on a port/hostname that doesn’t match what the backend is actually using.

check container is up:docker ps
docker logs service-catalog_backend_1
docker logs service-catalog_db_1

then i run :cat docker-compose.yml

The Problem: Network Isolation
Look closely at your networks configuration in docker-compose.yml:

backend is connected only to nginx-backend-net.
db is connected only to backend-db-net.

vim docker-compose.yml
and add :  backend:
    image: service-catalog:latest
    build:
      context: ./backend
    restart: unless-stopped
    environment:
      DATABASE_URL: "postgresql+psycopg2://catalog:catalog@db:5432/catalog"
    deploy:
      replicas: 1
    depends_on:
      - db
    networks:
      - nginx-backend-net
      - backend-db-net  # <--- ADD THIS LINE
      
 then :cat ./nginx/nginx.conf
 
Incorrect Hostname: Your Nginx config is trying to reach http://backend-api, but your Docker Compose service is named simply backend.
Evidence: Your ping backend command worked, but your Nginx config uses set $backend_upstream http://backend-api....
Incorrect Port: Your Nginx config is trying to use port 8080, but your backend logs clearly show the application is listening on port 5000.
Evidence: [INFO] Listening at: http://0.0.0.0:5000

to fix it .i change set $backend_upstream http://backend-api:8080;
 to 
 set $backend_upstream http://backend:5000;
the curl give 200



