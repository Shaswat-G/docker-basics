we will learn how to create our own images using a Dockerfile. A Dockerfile is a text file that contains instructions on how to build a Docker image. It specifies the base image, the commands to run, and any additional files or configurations needed for the image. By creating our own images, we can customize our applications and ensure consistency across different environments

We will save, load, and share our images using Docker Hub, a cloud-based registry for Docker images. This allows us to easily distribute our applications and collaborate with others. By the end of this section, you will have a solid understanding of how to build and manage your own Docker images.

We will optimize our images by minimizing the number of layers and using multi-stage builds. This will help reduce the size of our images and improve performance. We will also learn how to use .dockerignore files to exclude unnecessary files from being included in our images, further optimizing them.

An image contains all the necessary files to run an application including. Passive entity.:
1. cut down OS (like alpine, debian, ubuntu, etc.): Alpine is a popular choice for a base image since it is lightweight (30-40mb compared to 300-400mb for debain and ubuntu). It does not ship with bash, but it does have sh. Also, container images may be registered elsewhere like Azure Container Registry, Google Container Registry, etc. but Docker Hub is the most popular one.
2. third party dependencies (like node, python, etc.)
3. application source code (like index.js, package.json, etc.)
4. configuration files and environment variables (like .env, config.json, etc.)
5. commands to run the application (like CMD, ENTRYPOINT, healthcheck with build arguments, etc.)
6. Exposed ports (like EXPOSE, etc.)
7. External Volumes (like VOLUME, etc.)

A container is a runtime instance of an image. It is a lightweight, standalone, and executable package that includes everything needed to run an application. When we run a container, it creates a new instance of the image and executes the specified commands. Containers are isolated from each other and the host system, allowing for consistent and reproducible environments. They can be started, stopped, and deleted without affecting the underlying image. Its a special kind of process with its own filesystem, network, and isolated process tree.


Syntax:
FROM <base_image>
WORKDIR /app
COPY . .  <-- this is the same as ADD . . but COPY is preferred since it is more efficient and does not have the ability to extract tar files or download files from URLs. It simply copies files from the host to the image.
ADD . . 
RUN npm install <-- same commands we run in the terminal to install dependencies
RUN apt install python3 
ENV NODE_ENV=production  <-- this sets an environment variable in the image that can be accessed by the application at runtime.
EXPOSE 3000  <-- this is used to specify the port that the application will listen to, but we have to properly map container ports to host ports when running the container using the -p flag (like -p 3000:3000) to access the application from outside the container.

RUN addgroup -S appgroup && adduser -S appuser -G appgroup  <-- this creates a new user and group in the image to run the application with non-root privileges, which is a best practice for security. We can then use the USER instruction to switch to this user before running the application.

USER appuser

CMD ["node", "index.js"]  <-- this is the command that will be executed when the container starts. It is recommended to use the exec form (JSON array) instead of the shell form (string) to avoid issues with signal handling and to ensure that the command is executed directly without being passed through a shell.


run is a buildtime instruction that executes a command during the image build process. It is used to install dependencies, set up the environment, and perform any necessary configuration. The commands specified in the RUN instruction are executed in a new layer on top of the previous layers, and the resulting changes are committed to the image. This allows us to create a layered image that can be reused and shared efficiently.
cmd is a runtime instruction that specifies the command to be executed when a container is started from the image. It defines the default command that will run when the container is launched, but it can be overridden by providing a different command when running the container. The CMD instruction is used to specify the main process of the application that will run inside the container.


Speeding up builds:
1. Use a smaller base image (like alpine instead of debian or ubuntu)
2. Minimize the number of layers by combining commands into a single RUN instruction (like RUN apt update && apt install -y python3)
3. Use multi-stage builds to separate the build and runtime stages, which allows us to only include the necessary files and dependencies in the final image.


Since an image is sequentially composed of layers, docker provides layer caching to speed up subsequent builds. When we build an image, Docker checks if there are any existing layers that match the instructions in the Dockerfile. If a layer already exists, Docker will reuse it instead of rebuilding it, which can significantly reduce the build time. However, if any instruction in the Dockerfile changes, all subsequent layers will be invalidated and rebuilt, which can increase the build time. Therefore, it is important to structure our Dockerfile in a way that minimizes changes to the layers that are **stable** (like installing dependencies) and keeps the layers that are more likely to change (like copying source code) towards the end of the Dockerfile.


While building the image, this directory context is sent to the Docker daemon. The Docker daemon will look for a Dockerfile in the context and execute the instructions in the Dockerfile to build the image. The context can be a local directory or a remote repository. It is important to keep the context as small as possible to reduce the build time and avoid sending unnecessary files to the Docker daemon. We can use a .dockerignore file to exclude files and directories from being included in the context, similar to a .gitignore file. This helps optimize our builds and keeps our images lean. This is specified in .dockerignore file.

we use the following commands:
docker build -t <image_name>:<tag> .  <-- this command builds the image using the Dockerfile in the current directory (.) and tags it with the specified image name and tag. The -t flag is used to specify the image name and tag. If no tag is specified, it defaults to "latest". For example, docker build -t myapp:1.0 . will build an image named "myapp" with the tag "1.0" using the Dockerfile in the current directory.

Since we test and build dockerfile incrementally, it is useful to run the container in interactive mode to see the changes we made to the image. We can use the following command:
docker run -it <image_name>:<tag> /bin/bash  <-- this command runs a container from the specified image and tag in interactive mode (-it) and opens a bash shell inside. We can then inspect the filesystem and run commands. 