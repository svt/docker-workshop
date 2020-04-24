<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD024 -->
<!-- markdownlint-disable MD026 -->

# Chapter 2: The Dockerfile

In this chapter you will learn about Dockerfiles and how to use them to build your own images.

## Contents

1. [Where images come from](#where)
2. [Building images](#building)
3. [EXPOSE yourself](#expose)
4. [Widening our network](#network)
5. [More things to try](#more)

## 2.1 Where images come from <a name="where"></a>

In the last chapter we used *Docker images* to run containers. But where do the images come from?

Images are *built* using instructions provided in a *Dockerfile*. The Dockerfile is like a recipe for making Docker images.

So, from the top down: *Docker containers* are based on *Docker images*, which in turn are built from instructions in a *Dockerfile*.

### Anatomy of a Dockerfile

This is what a basic Dockerfile looks like:

```Dockerfile
FROM node:10-alpine AS build-stage

COPY ./package.json  /app/package.json
COPY ./package-lock.json  /app/package-lock.json
COPY ./blog /app/blog
COPY ./docs /app/docs
COPY ./src /app/src
COPY ./static /app/static
COPY ./sidebars.js /app/sidebars.js
COPY ./docusaurus.config.js /app/docusaurus.config.js
COPY ./build-search-data.js /app/build-search-data.js
COPY ./yarn.lock /app/yarn.lock

WORKDIR /app

RUN yarn
RUN yarn build
RUN node build-search-data.js

# Add the generated html to the ngnix image
FROM nginx:alpine AS deploy-stage
COPY --from=build-stage /app/build /usr/share/nginx/html

```

The Dockerfile is a structured set of instructions for how to create a docker image. Instructions ususally contains a base image, one or more actions that need to be performed to customize the image, and specifications for what processes the container based on the image should actually run. The general format of the file is:

```Dockerfile

# Comments
INSTRUCTION arguments
```

#### FROM - the base image

```Dockerfile
FROM ubuntu:latest
```

The `FROM` instruction specifies which pre-existing image the one we are building should be based on. It could be a public image, a simple OS image, a highly specified image optimized for running a certain type of code, or whatever suits us, really.

#### Adaptations

If we just wanted to run the base image, we wouldn't need a Dockerfile - we could just run the image directly. The Dockerfile is what we use when we want to adapt the base image to our needs. There are several instructions available for this purpose - a complete reference can be found in the [official documentation](https://docs.docker.com/engine/reference/builder/).

##### RUN

The `RUN` instruction lets us run any command we want as part of the build process. **NB: This is not the same as having the command run when the container runs**, but will be run as a build step only. A very common use for `RUN` is installing software packages that we want to have available in our new image:

```Dockerfile
RUN apt-get update && apt-get install -y curl
```

The example installs the `curl` software package using `apt-get`, the package manager for Debian and Ubuntu Linux. The `-y` switch tells apt-get to NOT wait for us to confirm the package selections before installing, since the default behaviour is to wait for our input - and we won't be able to giva any input during the build process. Keep this in mind for other `RUN` instructions as well - ensure that the entire command should be able to finish without any further input from the user.

There can be any number of `RUN` instructions in a Dockerfile, but it is considered good practice to chain like commands together. Why do you think that is?

<details>
<summary>Hint</summary>
Basically, the more instructions in the Dockerfile, the larger the image. This is due to the layered architecture of the image, which you can find out more about [here](https://dzone.com/articles/docker-layers-explained) if you want. The point to remember is this: Fewer RUN instructions make smaller images!
</details>

##### COPY

`COPY` does exactly what it sounds like: It copies files. Specifically, it copies files to the image's file system so that they are available inside any container based on the image. The COPY instruction takes a *relative* source path and an *absolute* target path:

```Dockerfile
# Copy the "src" directory in the working folder to the root of the image filesystem
COPY ./src /src
```

#### What to do

After we have adapted the image to our needs we still need to tell it what to do when a container starts! There are two instructions available to do this. They have the same syntax but interact differently with the `docker run` command.

##### CMD

`CMD` takes a list as an argument, and when the container starts will run the command that results from putting all the elements in the list together:

```Dockerfile
CMD ["my", "cool", "command"]
```

will run the command:

```sh
my cool command
```

and

```Dockerfile
CMD ["program", "--config-parameter", "value"]
```

will run the command:

```sh
program --config-parameter value
```

**Commands specified with `CMD` are overrun by commands given in the `docker run` command.** That is, if you specify `docker run my_image /bin/bash`, the container will **only** run `/bin/bash`, the command given on the command line, and the command given by the `CMD` instruction will **not** run when the container is started.

##### ENTRYPOINT

`ENTRYPOINT` is specified in the same way:

```Dockerfile
ENTRYPOINT ["my-service", "--some-option", "value"]
```

will run the command:

```sh
my-service --some-option value
```

Entrypoints, however, are **not** overrun by command line options. Instead, any commands given in the `docker run` command will be *appended* to the command defined in the `ENTRYPOINT`!

So assuming the same entrypoint instruction as above, running the container using the command

```sh
docker run my_image another-command
```

will make the container start with the command

```sh
my-service --some-option value another-command
```

As you've probably realized, this can be both problematic and very useful.

A note on the specification format: As you can see in the official documentation, both `CMD` and `ENTRYPOINT` can also be specified as a string instead of a list. This might look simpler and more intuitive, but has some repercussions primarily on how the resulting container accepts and handles signals that are sent to it. If you don't *know* that you want a specific behaviour that can only be acheived using the string format, stick with the list as it will handle signals in the proper way. More about command definition formats and signalling [here](https://hynek.me/articles/docker-signals/).

##### The whole picture

As you may already have figured out, the command run by the container when it starts is actually built up from both the `ENTRYPOINT` and the `CMD` instructions. The first part comes from the `ENTRYPOINT` if it is specified, and is not overreideable, and the second part is constructed from the `CMD` (if present) and can be overwritten by specifying a different command in `docker run`.

Together they can be used to build very flexible images whose behaviour can be controlled in great detail. A common practice is to combine the two like so:

```Dockerfile
ENTRYPOINT ["executable"]
CMD ["option1", "option2"]
```

This locks the container to running `executable` on upstart and provides default options for the `executable` program that can be overridden from the `docker run` command.

So, these three examples do the same thing per default, but allow for different levels of overriding:

```Dockerfile
CMD [ "program", "--option1", "option1-value", "--option2", "option2-value" ]
# This can be completely overridden on the command line
```

```Dockerfile
ENTRYPOINT [ "program", "--option1", "option1-value" ]
CMD [ "--option2", "option2-value" ]
# Here, --option2 and it's value can be overridden or replaced by something else.
```

```Dockerfile
ENTRYPOINT [ "program" ]
CMD [ "--option1", "option1-value", "--option2", "option2-value" ]
# Here, both options can be overriden, but the container will always run "program"
```

```Dockerfile
ENTRYPOINT [ "program", "--option1", "option1-value", "--option2", "option2-value" ]
# This container will always run the program with the two options, and we can add more
# options on the command line if we want to.
```

### Practice: Create a Dockerfile

Create a new Dockerfile (named "Dockerfile") that builds an image with the following specifications:

* Base image: ubuntu:latest
* Installed packages: nginx
* What the container should run: `nginx -g "daemon off;"`

Ubuntu is an image providing a basic Ubuntu Linux OS and nginx is a popular web server package.

Consider what the best way to run the nginx-command might be. CMD, ENTRYPOINT or a combination of the two?

<details>
<summary>Hint</summary>

A working Dockerfile that complies with the specification might look like this:

```Dockerfile
FROM ubuntu:latest

RUN apt update && apt -y install nginx

ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
```

</details>

### More resources

* [Official `Dockerfile` reference](https://docs.docker.com/engine/reference/builder/)
* [Dockerfile tutorial by example](https://takacsmark.com/dockerfile-tutorial-by-example-dockerfile-best-practices-2018/)
* [CMD vs ENTRYPOINT](https://dev.to/lasatadevi/docker-cmd-vs-entrypoint-34e0)

---

<img src="img/the_big_idea.jpeg" alt="The big idea - Part of a fanzine by Julia Evans" />

---

## 2.2 Building images <a name="building"></a>

Once you have a Dockerfile, you can start cooking up some images! The command to build an image from a Dockerfile located in the current working directory is:

```sh
docker build .
```

There are more available parameters, but for now let's stick with the simplest way, which is:

* The dockerfile file name should be "Dockerfile"
* Stand in the same directory as your dockerfile
* Run `docker build .`

You should see output reflecting the build steps you defined in the previous step.

### Practice: Build a container image

From a terminal, move to the directory where you created the Dockerfile in the last exercise. Run the build command. Check the output for errors.

<details>
<summary>Hint</summary>
You may see errors if, for example, you referred a non-existing base image or if a command called with `RUN` could not complete successfully. The output will indicate the nature of the problem and which build step it was that failed. Of course, always check for typos first :)
</details>

#### How you'll know it worked

When the build command finishes successfully you should be able to see a new local image with `docker images`:

```sh
$ docker images
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
<none>                     <none>              25c097a6d2d0        5 seconds ago       152MB
ubuntu                     latest              72300a873c2c        6 days ago          64.2MB
...
```

We can see a brand new image at the top of the list! Just a few seconds old, so it must be the one we just built. But it looks a bit...ugly. No name, no tag, completely unidentifiable.

### Practice: Name the image

The `docker build` command has a `-t` switch that lets you specify a name and tag for the image being built. The syntax is:

```sh
docker build -t <name>:<tag> .
```

Use it to build the image again, this time using a nice name and tag that you pick for your image. Does the build take the same amount of time as before?

<details>
<summary>Hint</summary>
The build will finish in an instant if you have not made any changes to the Dockerfile between builds. Each successive build will make use of the cache of previously built layers as far as possible and will only re-build the image from the first changed layer up. Our new build didn't need to change anything but the name and tag, so no parts of the image really needed rebuilding!
</details>

#### How you'll know it worked

The output from `docker images` will show your brand new name and tag instead of `<none>`. Nice!

There is another command, `docker tag`, that is specifically for working with image tags. You can use it to create alternative tags for your image:

```sh
docker tag <current name>:<current tag> <new name:new tag>
```

### Practice: More tags!

Use the `docker tag` command to create another tag of you choice for your image. You can create a new name too if you want!

<details>
<summary>Hint</summary>

The command should look something like this:

```sh
docker tag myimage:latest myimage:stable
```

</details>

#### How you'll know it worked

You should now see both name/tag combinations listed in the output of `docker images`. Notice that the ID is the same for both of them!

The ID is the reference that docker itself uses to identify the image, the names and labels are just shortcuts. This also means that regardless of how many tags and names you stick to it, the image will still only be stored once per build version.

### More resources

* [The `docker build` command reference](https://docs.docker.com/engine/reference/builder/)
* [The `docker tag` command reference](https://docs.docker.com/engine/reference/commandline/tag/)
* [A quick introduction to Docker tags](https://www.freecodecamp.org/news/an-introduction-to-docker-tags-9b5395636c2a/)

---

## 2.3 EXPOSE yourself <a name="expose"></a>

We need just one more thing in order to turn our little nginx-running container into a complete web server: an `EXPOSE` instruction, that tells the container that a specific port is available to the network:

```Dockerfile
EXPOSE <port>
```

### Practice: Expose a web server port

Edit the Dockerfile again and add an `EXPOSE` instruction to exposr *port 80* just before your `CMD` or `ENTRYPOINT`. Then rebuild the container with an appropriate name and label.

<details>
<summary>Hint</summary>

The new line should read:

```Dockerfile
EXPOSE 80
```

</details>

Now we should have a working web server in a container! Let's try to run it! Use `docker run` and give the name and tag of the last image you built. Then check what's going on using `docker ps`.

```sh
$ docker ps
CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS              PORTS               NAMES
8104dd477e15        mywebserver:latest   "nginx -g 'daemon of…"   12 seconds ago      Up 12 seconds       80/tcp              confident_shirley
```

Under "PORTS" we now see port 80 listed! Cool. Docker even gave the running container a silly nickname, listed in the last column. Very professional.

### Practice: Check that the web server is accessible

Now you should be able to point a browser to the running container. Remember how we found out the IP address of a container at the end of [chapter 1](../1-basics/README.md)?

#### How you'll know it worked

If you point your browser to the container's IP address, you should be able to enjoy Nginx's lovely default index page. Now check the terminal where you started the container again. What is that output?

### Actually...

...that whole thing wasn't *strictly* necessary. The `EXPOSE` command doesn't really *do* anything with the container, except show the port in the `docker ps` output. In fact, try removing it and pointing your browser to the container's IP again - it should still work. But you can certainly see how *making information about what ports a container uses* very explicit and visible can be useful. It is good practice to do so and it will help you *a lot* when working with multiple containers.

In short: **Always tell the world what ports your containers need with `EXPOSE`, even though you don't need to for things to work.**

### More resources

* [`EXPOSE` instruction reference](https://docs.docker.com/engine/reference/builder/#expose)
* [Docker expose ports](https://vsupalov.com/docker-expose-ports/)

---

## 2.4 Widening our network <a name="network"></a>

The automatically assigned private IP of the container has a problem though: We are the only ones that can access it. Most of the time, we want the thing we run in containers to look **as if** they were running directly on our computer, network-wise. This is done via additional instructions to `docker run`.

```sh
docker run -p <host_port>:<container_port> <imagename>:<tag>
```

The `-p` option lets us specify a port on our computer that should be bound to a specific port on the container.

### Practice: Bind to a port on your computer

Stop the running container and then start it again, this time using the `-p` option to bind the containers exposed port to your computer's port **8080**.

<details>
<summary>Hint</summary>

The command should look something like this:

```sh
docker run -p 8080:80 mynginximage:latest
```

</details>

#### How you'll know it worked

[This link](http://localhost:8080) should now show the Nginx default index page!

### More resources

* [The publish port option specification](https://docs.docker.com/engine/reference/commandline/run/#publish-or-expose-port--p---expose)
* [Binding docker ports](https://runnable.com/docker/binding-docker-ports)

## 2.5 More things to try <a name="more"></a>

By now you should know enough to build and run pretty much any container you want (with the help of the documentation, at least). You might want to try one or more of the following:

* Replace the nginx default index page with a custom page that you created.
* Explore different base OS images. What software is available in them? There is a whole ecosystem of images out there catering to different development languages. What seems to be the most used images for the language/s you usually work with? What makes them well suited?
* Look at the Dockerfiles for some official images in the Docker Hub. Do you understand what they do? Look up any unfamiliar instructions you find in the [Dockerfile reference](https://docs.docker.com/engine/reference/builder/).
* Select a software project that you understand well and try to containerize it yourself. If there is already an official Docker image for it, don't look at it before you start! Use it for peeking if you run out of ideas.

Good job! When you feel ready, move on to [Chapter 3](../3-dynamics/README.md) where the magic really starts to happen.
