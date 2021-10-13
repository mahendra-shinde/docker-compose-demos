# Docker Compose with Load Balancer (NGINX)

1. Create a folder 'demo3' and create a subfolder 'nginx'

    ```
    $ cd C:
    $ mkdir \demo3
    $ cd \demo3
    $ mkdir nginx
    ```

2.  Create a configuration file [nginx.conf](./nginx/nginx.conf) for Load balancer configuration

    ```yml
    user nginx;
    worker_processes    auto;
    events { worker_connections 1024; }
    http {
        include             /etc/nginx/proxy.conf;
        include             /etc/nginx/mime.types;
        limit_req_zone      $binary_remote_addr zone=one:10m rate=5r/s;
        server_tokens       off;
        sendfile            on;
        keepalive_timeout   29; # Adjust to the lowest possible value that makes sense for your use case.
        client_body_timeout 10; client_header_timeout 10; send_timeout 10;

        upstream webapp {
            ## Must match with "Service name" in "docker-compose.yaml"
            server          app:80;
        }

        server {
            listen          80;
            server_name     $hostname;

            location / {
                proxy_pass  http://webapp;
                limit_req   zone=one burst=10 nodelay;
            }
        }
    }
    ```

3.  Create another [proxy](./nginx/proxy.conf) config file.

    ```yml
    ## nginx reverse proxy configuration
    proxy_redirect          off;
    proxy_http_version      1.1;
    proxy_set_header        Upgrade             $http_upgrade;
    proxy_cache_bypass      $http_upgrade;
    proxy_set_header        Connection          keep-alive;
    proxy_set_header        Host $host;
    proxy_set_header        X-Real-IP           $remote_addr;
    proxy_set_header        Cache-Control       no-cache;
    proxy_set_header        X-Forwarded-For     $proxy_add_x_forwarded_for;
    proxy_set_header        X-Forwarded-Proto   $scheme;
    proxy_set_header        X-Forwarded-Host    $server_name;
    client_max_body_size    10m;
    client_body_buffer_size 128k;
    proxy_connect_timeout   90;
    proxy_send_timeout      90;
    proxy_read_timeout      90;
    proxy_buffers           32 4k;
    ```

4.  Save both files inside 'nginx' subfolder.

5.  Create a new [docker-compose.yml](./docker-compose.yml) in 'demo3' folder.

    ```yml
    version: "3.0"
    services:
        lb:
            image: nginx:alpine
            hostname: nginx
            volumes:
            - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
            - ./nginx/proxy.conf:/etc/nginx/proxy.conf:ro
            ## Capture all server logs at 'logs' directory
            - ./nginx/logs/:/var/log/nginx/
            ports:
            ## Set port forwarding to local port 8080 from nginx port 80
            - 8080:80
            depends_on:
            - app

        app:
            image: mahendrshinde/myweb
            ports:
            - 80
            deploy:
               replicas: 3
    ```
  
6.  Launch the application and test the ports

    ```
    $ cd \demo3
    $ docker-compose up -d
    $ docker-compose ps
    $ wget http://localhost:8080
    ```

7.  Clean-Up

    ```
    $ docker-compose down
    ```

## Deploy on Docker Swarm at Playground

1.  Visit Docker Playground `https://labs.play-with-docker.com`
2.  Login using Docker Credentials
3.  Click "Start" button
4.  Click the configure button and choose template `3 Managers & 2 workers`
5.  In side terminal run following commands:

  ```
  $ git clone https://github.com/mahendra-shinde/docker-compose-demos
  $ cd docker-compose-demos/demo2
  $ docker-compose config
  $ docker stack deploy app1 --compose-file docker-compose.yml
  $ docker stack ls 
  $ docker stack ps app1
  ## Undeploy 
  $ docker stack rm app1
  ```