# Simple Django project with docker, docker-compose and CI/CD using mikr.us  
## Intro
This repository was created for the speech at the [PyStok](https://pystok.org/pystok-55/) [#55](https://www.facebook.com/events/486108750034527) event. 
This guide will demonstrate how to use Docker, Docker-Compose, and Gitlab CI to make your small project Dockerize and deploy to [mikr.us](https://mikr.us/) VPS.

## Requirements
Before start make sure that you have installed:
- [Docker](https://docs.docker.com/desktop/) ([Linux](https://docs.docker.com/desktop/install/linux-install/), [Mac](https://docs.docker.com/desktop/install/mac-install/), [Windows](https://docs.docker.com/desktop/install/windows-install/))
- [Docker Compose v2](https://docs.docker.com/compose/install/) - You need this only if you have installed Docker Engine or Docker CLI. Docker Desktop includes Docker Compose.

## Creating a project from scratch
#### For a better understanding, I encourage you to read:
 - [Docker tutorial](https://docs.docker.com/get-started/)
 - [Docker Compose tutorial](https://docs.docker.com/compose/gettingstarted/)
   - [Explore the full list of Compose commands](https://docs.docker.com/compose/reference/)
   - [Explore the Compose configuration file reference](https://docs.docker.com/compose/compose-file/)
 - [Django tutorial](https://docs.djangoproject.com/en/3.2/intro/tutorial01/)

### Let's start with our project!

1. Create an empty project directory.
    
    ```console
    mkdir django-poll
    ```

    You can name the directory something easy for you to remember. This directory is the context for your application image. The directory should only contain resources to build that image.

2. Create a new file called `Dockerfile` in your project directory.

    The Dockerfile defines an application's image content via one or more build
    commands that configure that image. Once built, you can run the image in a
    container. For more information on `Dockerfile`, see the [Docker user guide](https://docs.docker.com/get-started/)
    and the [Dockerfile reference](https://docs.docker.com/engine/reference/builder/).

3. Add the following content to the `Dockerfile`.

    ```dockerfile
    # syntax=docker/dockerfile:1
    FROM python:3
    ENV PYTHONDONTWRITEBYTECODE=1
    ENV PYTHONUNBUFFERED=1
    WORKDIR /code
    COPY requirements.txt /code/
    RUN pip install -r requirements.txt
    COPY . /code/
    ```

    This `Dockerfile` starts with a [Python parent image](https://hub.docker.com/r/library/python).
    The parent image is modified by:
    - Adding [Custom Dockerfile syntax](https://docs.docker.com/build/buildkit/dockerfile-frontend/
    - settings environment variables:
      - `ENV PYTHONDONTWRITEBYTECODE=1` - [This prevents Python from writing out pyc files](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONDONTWRITEBYTECODE)
      - `ENV PYTHONUNBUFFERED=1` - [This keeps Python from buffering stdin/stdout](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONUNBUFFERED)
      - `WORKDIR /code` - adding a new `code` directory
      - `COPY requirements.txt /code/` - coping `requirements.txt` file from [context](https://docs.docker.com/build/building/context/#path-context) to `code` folder inside docker image
      - `RUN pip install -r requirements.txt` - installing the Python requirements defined in the `requirements.txt` file

4. Create a `requirements.txt` in your project directory.

    This file is used by the `RUN pip install -r requirements.txt` command in your `Dockerfile`.

5. Add the required software in the file.

    ```python
    # Install last Django LTS
    Django>=3.2,<3.3
    # Install postgresql database adapter
    psycopg2>=2.9
    ```
   
6. Create a file called `docker-compose.yml` in your project directory.

    The `docker-compose.yml` file describes the services that make your app. In
    this example those services are a web server and database.  The compose file
    also describes which Docker images these services use, how they link
    together, any volumes they might need to be mounted inside the containers.
    Finally, the `docker-compose.yml` file describes which ports these services
    expose. See the [`docker-compose.yml` reference](https://docs.docker.com/compose/compose-file/) for more
    information on how this file works.

7. Add the following configuration to the file.

    ```yaml
    services:
      db:
        image: postgres
        volumes:
        - postgresql_data:/var/lib/postgresql/data
        environment:
        - POSTGRES_PASSWORD=${POSTGRES_PASSWORD:-postgres}
      web:
        build: .
        command: python manage.py runserver 0.0.0.0:8000
        volumes:
        - .:/code
        ports:
        - "8000:8000"
        environment:
        - POSTGRES_PASSWORD=${POSTGRES_PASSWORD:-postgres}
        depends_on:
        - db
    
    volumes:
      postgresql_data:
    ```

    This file defines two services: The `db` service and the `web` service.\
    `db` service will mount `postgresql_data` [volume](https://docs.docker.com/storage/volumes/) to 
    `/var/lib/postgresql/data` inside docker. Thanks to this, data from the container won't be lost. We also set new
    environment variable: `POSTGRES_PASSWORD` with password to postgres database. `${POSTGRES_PASSWORD:-postgres}` means
    take environment variable `POSTGRES_PASSWORD` or if it doesn't exist set default value to `postgres`.


### Create a Django project

In this step, you create a Django starter project by building the image from the build context defined in the previous procedure.

1. Go to the root of your project directory.
    ```console
    cd django-poll
    ```

2. Create the Django project by running the [docker compose run](https://docs.docker.com/engine/reference/commandline/compose_run/)
   command as follows. Project name should follow [PEP-8](https://peps.python.org/pep-0008/#package-and-module-names) naming convention.

   ```console
   docker compose run web django-admin startproject django_poll .
   ```

   This instructs Compose to run `django-admin startproject <project_name>`
   in a container, using the `web` service's image and configuration. Because
   the `web` image doesn't exist yet, Compose builds it from the current
   directory, as specified by the `build: .` line in `docker-compose.yml`.

   Once the `web` service image is built, Compose runs it and executes the
   `django-admin startproject` command in the container. This command
   instructs Django to create a set of files and directories representing a
   Django project.

3. After the `docker compose` command completes, list the contents of your project.

   ```console
   $ ls -l

    drwxr-xr-x 2 root    root    django_poll
    -rw-rw-r-- 1 user    user    docker-compose.yml
    -rw-rw-r-- 1 user    user    Dockerfile
    -rwxr-xr-x 1 root    root    manage.py
    -rw-rw-r-- 1 user    user    requirements.txt

   ```

   If you are running Docker on Linux, the files `django-admin` created are
   owned by root. This happens because the container runs as the root user.
   Change the ownership of the new files.

   ```console
   sudo chown -R $USER:$USER django_poll manage.py
   ```

   If you are running Docker on Mac or Windows, you should already
   have ownership of all files, including those generated by
   `django-admin`. List the files just to verify this.

## Credits and more documentation:
* [Docker overview](https://docs.docker.com/get-started/)
* [Docker Compose overview](https://docs.docker.com/compose/)
* [Awesome Compose Django sample application](https://github.com/docker/awesome-compose/tree/master/official-documentation-samples/django/)
* [Docker overview](https://docs.djangoproject.com/en/3.2/intro/overview/)