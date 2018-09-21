# What is Docker?
* Docker is a tool designed to make it easier to create, deploy, and run applications by using containers.
* Docker allows applications to use the same Linux kernel as the system that they're running on.

# What is a Container?
* Containers allow a developer to package up an application with all of the dependencies and ship it all out as one package.

# Purpose of Repo
This repo is a Hello World example of running an application with Docker.
* Note: I assume that Docker has been installed already.

## List Docker commands
* docker <br/>
* docker container --help
* docker --version <br/>
* docker version <br/>
* docker info
* docker run hello-world
* docker image ls
* docker container --all

## Dockerfile
* Defines what goes on in the environment inside your container
* Access to resources like networking, disk drive, etc..

## [How to]
1. Create empty directory
2. Create a file called **Dockerfile**, example: <br/>
    ```python
    # Use an official Python runtime as a parent image
    FROM python:2.7-slim
    
    # Set the working directory to /app
    WORKDIR /app
    
    # Copy the current directory contents into the container at /app
    ADD . /app
    
    # Install any needed packages specified in requirements.txt
    RUN pip install --trusted-host pypi.python.org -r requirements.txt
    
    # Make port 80 available to the world outside this container
    EXPOSE 80
    
    # Define environment variable
    ENV NAME World
    
    # Run app.py when the container launches
    CMD ["python", "app.py"]
    ```
3. Create two more files: requirements.txt and app.py <br/>
 * requirements.txt <br/>
    ```txt
    Flask
    Redis
    ```
 * app.py
    ```python
    from flask import Flask
    from redis import Redis, RedisError
    import os
    import socket

    # Connect to Redis
    redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

    app = Flask(__name__)

    @app.route("/")
    def hello():
        try:
            visits = redis.incr("counter")
        except RedisError:
            visits = "<i>cannot connect to Redis, counter disabled</i>"

        html = "<h3>Hello {name}!</h3>" \
               "<b>Hostname:</b> {hostname}<br/>" \
               "<b>Visits:</b> {visits}"
        return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=vistits)

    if __name__ == "__main__":
        app.run(host='0.0.0.0', port=80)
    ```
4. Create the Docker image by running the following command within the app directory: <br/>
    ```bash
    docker build -t friendlyhello .
    ```
5. Where is your build image? It's in your machine's local Docker image registry: <br/>
    ```bash
    docker image ls
    ```
6. Run the app in background: <br/>
    ```bash
    docker run -d -p 4000:80 friendlyhello
    ```
7. View results
 * HTML view by opening webpage http://localhost:4000
 * Bash view <br/>
    ```bash
    curl http://localhost;4000
    ```
8. Shutdown the container: <br/>
    ```bash
    docker container ls
    docker container stop <CONTAINER_ID>
    ```

# Services

## About Services
Services are just __containers in production__. Codifies the way that images run
* What ports to use
* How many replicas of the container should run
Services are defined by the docker-compose.yml file
* A YAML file that defines how Docker containers should behave in production

## Your first `docker-compose.yml` 
* Create the `docker-compose.yml`
    ```yaml
    version: "3"
    services:
      web:
        # replace username/repo:tag with your name and image details
        image: username/repo:tag
        deploy:
          replicas: 5
          resources:
            limits:
              cpus: "0.1"
              memory: 50M
          restart_policy:
            condition: on-failure
        ports:
          - "4000:80"
        networks:
          - webnet
      networks:
        webnet:
    ```


[How to]: https://docs.docker.com/get-started/part2/#your-new-development-environment
