# Example project from [A First Look at Docker](https://dev.to/ajcwebdev/a-first-look-at-docker-3hfg)

[Docker](https://www.docker.com/) is a set of tools that use OS-level virtualization to deliver software in isolated packages called containers. Containers bundle their own software, libraries and configuration files. They communicate with each other through well-defined channels and use fewer resources than virtual machines.

## Outline

* Node project with an Express server
  * Run server
* Container image
  * `Dockerfile`
  * `.dockerignore`
* Build image
  * Build project with `docker build`
  * List Docker images with `docker images`
* Run image
  * Run Docker container with `docker run`
  * List containers with `docker ps`
  * Print output of your app with `docker logs`
  * Call your app using `curl`
* `docker-compose.yml` file
  * Create and start containers with `docker compose up`
* Push your project to a GitHub repository
  * Create a new blank repository
  * Create `.gitignore` file
  * Initialize Git and push to newly created repo
* Publish to GitHub Container Registry
  * Login to `ghcr.io` with `docker login` 
  * Tag image
  * Push to registry
  * Pull your image

## 1. Node project with an Express server

We have a simple Node application with Express that returns an HTML fragment.

```javascript
// index.js

const express = require("express")
const app = express()

const PORT = 8080
const HOST = '0.0.0.0'

app.get('/', (req, res) => {
  res.send('<h2>ajcwebdev-docker</h2>')
})

app.listen(PORT, HOST)
console.log(`Running on http://${HOST}:${PORT}`)
```

### Run server

```bash
node index.js
```

## 2. Container image

You'll need to build a Docker image of your app to run this app inside a Docker container using the official Docker image. We will need two files: `dockerfile` and `.dockerignore`.

### Dockerfile

Docker can build images automatically by reading the instructions from a [`Dockerfile`](https://docs.docker.com/engine/reference/builder/). A `Dockerfile` is a text document that contains all the commands a user could call on the command line to assemble an image. Using `docker build` users can create an automated build that executes several command-line instructions in succession.

### FROM

```dockerfile
FROM node:14-alpine
```

The `FROM` instruction initializes a new build stage and sets the Base Image for subsequent instructions. A valid `Dockerfile` must start with `FROM`. The first thing we need to do is define from what image we want to build from. We will use version `14-alpine` of `node` available from [Docker Hub](https://hub.docker.com/_/node) because the universe is chaos and you have to pick something so you might as well pick something with a smaller memory footprint.

### WORKDIR

```dockerfile
WORKDIR /usr/src/app
```

The `WORKDIR` instruction sets the working directory for your application to hold the application code inside the image. 

### COPY

```dockerfile
COPY package*.json ./
```

This image comes with Node.js and NPM already installed so the next thing we need to do is to install your app dependencies using the `npm` binary. The `COPY` instruction copies new files or directories from `<src>`. The `COPY` instruction bundles your app's source code inside the Docker image and adds them to the filesystem of the container at the path `<dest>`.

### RUN

```dockerfile
RUN npm i
COPY . ./
```

The `RUN` instruction will execute any commands in a new layer on top of the current image and commit the results. The resulting committed image will be used for the next step in the `Dockerfile`. Rather than copying the entire working directory, we are only copying the `package.json` file. This allows us to take advantage of cached Docker layers.

### EXPOSE

```dockerfile
EXPOSE 8080
```

The `EXPOSE` instruction informs Docker that the container listens on the specified network ports at runtime. Your app binds to port `8080` so you'll use the `EXPOSE` instruction to have it mapped by the `docker` daemon.

### CMD

```dockerfile
CMD ["node", "index.js"]
```

Define the command to run your app using `CMD` which defines your runtime. The main purpose of a `CMD` is to provide defaults for an executing container. Here we will use `node index.js` to start your server.

### Complete Dockerfile

Your `Dockerfile` should now look like this:

```dockerfile
FROM node:14-alpine
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm i
COPY . ./
EXPOSE 8080
CMD [ "node", "index.js" ]
```

### .dockerignore

Before the docker CLI sends the context to the docker daemon, it looks for a file named `.dockerignore` in the root directory of the context. Create a `.dockerignore` file in the same directory as your `Dockerfile`. If this file exists, the CLI modifies the context to exclude files and directories that match patterns in it. This helps avoid sending large or sensitive files and directories to the daemon.

```
node_modules
Dockerfile
.dockerignore
.git
.gitignore
npm-debug.log
```

## 3. Build image

Go to directory with `Dockerfile` and build the Docker image. The `-t` flag lets you tag your image so it's easier to find later using the `docker images` command:

### Build project with `docker build`

The [`docker build`](https://docs.docker.com/engine/reference/commandline/build/) command builds an image from a Dockerfile and a "context". A build’s context is the set of files located in the specified `PATH` or `URL`. The `URL` parameter can refer to three kinds of resources:
* Git repositories
* Pre-packaged tarball contexts
* Plain text files

```bash
docker build . -t ajcwebdev/ajcwebdev-docker
```

### List Docker images with `docker images`

Your image will now be listed by Docker. The [`docker images`](https://docs.docker.com/engine/reference/commandline/images/) command will list all top level images, their repository and tags, and their size.

```bash
docker images
```

## 4. Run the image

Docker runs processes in isolated containers. A container is a process which runs on a host. The host may be local or remote.

### Run the Docker container with `docker run`

When an operator executes [`docker run`](https://docs.docker.com/engine/reference/run/), the container process that runs is isolated in that it has its own file system, its own networking, and its own isolated process tree separate from the host.

```bash
docker run -p 49160:8080 -d ajcwebdev/ajcwebdev-docker
```

`-d` runs the container in detached mode, leaving the container running in the background. The `-p` flag redirects a public port to a private port inside the container.

### List containers with `docker ps`

To test your app, get the port of your app that Docker mapped:

```bash
docker ps
```

### Print the output of your app with `docker logs`

```bash
docker logs <container id>
```

### Call your app using `curl`

```bash
curl -i localhost:49160
```

## 5. `docker-compose.yml` file

[Compose](https://docs.docker.com/compose/) is a tool for defining and running multi-container Docker applications. After configuring your application’s services with a YAML file, you can create and start all your services with a single command. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.

```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "49160:8080"
```

### Create and start containers with `docker compose up`

The `docker compose up` command aggregates the output of each container. It builds, (re)creates, starts, and attaches to containers for a service.

```bash
docker compose up
```

## 6. Push your project to a GitHub repository

We can publish this image to the GitHub Container Registry with GitHub Packages. This will require pushing our project to a GitHub repository.

### Create a new blank repository

You can create a blank repository by visiting [repo.new](https://repo.new).

### Create `.gitignore` file

Add the following files to `.gitignore`.

```
node_modules
package-lock.json
yarn.lock
.DS_Store
.env
dist
build
*.local
```

### Initialize Git and push to newly created repo

```bash
git init
git add .
git commit -m "I can barely contain my excitement"
git branch -M main
git remote add origin https://github.com/ajcwebdev/ajcwebdev-docker.git
git push -u origin main
```

## 7. Publish to GitHub Container Registry

[GitHub Packages](https://docs.github.com/en/packages/learn-github-packages/introduction-to-github-packages) is a platform for hosting and managing packages that combines your source code and packages in one place including containers and other dependencies. GitHub's [Container registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry) is optimized for containers and supports Docker and OCI images.

### Login to `ghcr.io` with `docker login` 

To login, create a [PAT (personal access token)](https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token) with the ability to read, write, and delete packages and include it instead of `xxxx`.

```bash
export CR_PAT=xxxx
```

Login with your own username in place of `ajcwebdev`.

```bash
echo $CR_PAT | docker login ghcr.io -u ajcwebdev --password-stdin
```

### Tag image

```bash
docker tag ajcwebdev/ajcwebdev-docker ghcr.io/ajcwebdev/ajcwebdev-docker
```

### Push to registry

```bash
docker push ghcr.io/ajcwebdev/ajcwebdev-docker:latest
```

### Pull your image

To test that our project has a docker image published to a public registry, pull it from your local development environment.

```bash
docker pull ghcr.io/ajcwebdev/ajcwebdev-docker
```

You can view this published container [on my GitHub](https://github.com/ajcwebdev/ajcwebdev-docker/pkgs/container/ajcwebdev-docker).