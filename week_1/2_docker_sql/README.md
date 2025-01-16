[Docker Video Lesson](https://www.youtube.com/watch?v=EYNwNlOrpr0&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb&index=5)

# ENVIRONMENT SETUP

I am using WSL2 on Windows in order to run Docker Desktop. [This guide](https://learn.microsoft.com/en-us/windows/wsl/install) is helpful to get started.

Then, from the Ubuntu Terminal, install Anaconda to get access to and manage Python, Pip, Jupyter, and other handy Data Engineer tools.

# DOCKER

Allows you to run images which are packages containing OS, software, libraries, etc.

### BENEFITS

- Reproducable: Can create an image and deploy it to numerous different systems, with all dependancies in place.
- Isolation: If an error occurs, something is misconfigured, etc. in a running instance, no worries! Your original image still exists and you can use it to recover.

# DOCKERFILE

Text file used by Docker to create an image. A simple example:

>	FROM python:3.9
>
>	RUN pip install pandas
>
>	WORKDIR /app
>	COPY pipeline.py pipeline.py
>
>	ENTRYPOINT ["python", "pipeline.py"]

These are command line commands.

- FROM indicates what the base image will be
- RUN is a build command
- WORKDIR makes the default working directory of the image
- COPY adds a file to the working directory. The last arg will always be the destination, all prior args will be copied to that destination (in this case copying our pipeline file into the container's working directory and keeping the same name)
- ENTRYPOINT specifies commands to execute from within the container. In this case, run python, then execute our pipelin script

### TO BUILD CONTAINER

	docker build .

- builds the Docker container according to the Dockerfile
- can add an optional tag with -t for example

	docker build -t test:pandas .

### TO RUN CONTAINER

	docker run -it test:pandas [args]

- runs the container tagged as test:pandas in interactive mode (-it) with some optional args passed in

# POSTGRES

Docker has built-in Postgres container. We can run the container to set up a database and populate it with a dataset

### CREATING A DB

Made a bash script to set up the postgres container via Docker

```
docker run -it \
    -e POSTGRES_USER="root" \
    -e POSTGRES_PASSWORD="root" \
    -e POSTGRES_DB="ny_taxi" \
    -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
    -p 5432:5432 postgres:13
```

- -v is for mounting the volume. The host machine will have a dirctory mapped to the directory in the container.
- -p is the port argument. 5432 is the port on both the container and host machine