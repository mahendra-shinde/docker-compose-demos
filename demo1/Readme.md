# Docker-compose demo1

1. Create a new directory "demo1"

    ```
    mkdir c:\demo1
    cd \demo1
    ```

2.  Create a file 'docker-compose.yml' and open in VS Code
    > Replace 'code' with 'notepad' if VS Code installed.

    ```
    code docker-compose.yml
    ```

3.  Add lines from [this](./docker-compose.yml) file.

4.  Open command prompt or powershell prompt and use following commands

    ```
    $ cd c:
    $ cd \demo1
    $ docker-compose config
    # Check for ERRORs, if contents of file are displayed, that means NO-ERRORS
    $ docker-compose up -d
    # Wait for 2 minutes
    $ docker-compose ps
    ```

5.  Notice the ports shown by last command (Usually  3XXXX) use it to connect Web 
    eg http://localhost:32676/ for NGINX


6.  