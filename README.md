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
 - [Django tutorial](https://docs.djangoproject.com/en/4.1/intro/tutorial01/)

***Let's start with our project!***


## Credits and more documentation:
* [Docker overview](https://docs.docker.com/get-started/)
* [Docker Compose overview](https://docs.docker.com/compose/)
* [Awesome Compose Django sample application](https://github.com/docker/awesome-compose/tree/master/official-documentation-samples/django/)