[Docker Video Lesson](https://www.youtube.com/watch?v=EYNwNlOrpr0&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb&index=5)

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