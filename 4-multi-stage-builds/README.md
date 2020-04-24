<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD024 -->
<!-- markdownlint-disable MD026 -->

# Chapter 4: Multi-stage builds

Moving the actual building of your code into docker has several advantages - one of which is that you no longer need to have all build dependencies present on your computer in order to build. In this chapter you will try out multi-stage docker builds and create a Dockerfile that takes care of both building the code and running it.

## The problem

We'll revisit our example-app for a bit. Let's say we've made a change to the content, and now we want to build a new image with the updated content. Assuming we just cloned the repo and have no dependencies installed, we would potentially have to do several things:

1. Install node.js.
2. Install the node dependencies with `npm install`.
3. Rebuild the static html content in `_book` using `npm run build`.
4. Rebuild the docker image.

Feels like a lot of work to add another picture of a cute dog, right? And what if we don't want to install all those npm packages on our computer? Luckily, docker can do all the steps for us.

## The multi-stage build

A not so well-known secret of the Dockerfile is that you can use as many `FROM` instructions as you like. When you provide a second `FROM` instruction, the build simply switches to using another base image and continues executing any subsequent instructions in that one instead. So basically, we could do the building in one image, and the web serving in another!

Now would probably be a good time to read the [documentation on multi-stage Docker builds](https://docs.docker.com/develop/develop-images/multistage-build/).

It looks like we can copy content from a previous stage using an instruction like

```Dockerfile
COPY --from=<stage-name> /path/in/previous/stage /target/path/in/current/stage
```

### Try it

Modify the Dockerfile to a multi-stage build. The file should contain two stages: One that copies the relevant source files and runs the necessary build commands, and one the copies the generated static content to a web server image. Try using the `node:alpine`-image for the first stage, it is an image built specifically for node applications. Stick with the `nginx` image for the second part. Then update some of the content in the `content` folder, rebuild the app and run the new image to see your changes!

<details>

<summary>Hint1</summary>

Remember that in the build stage you need to copy all relevant files to the image before you can run the needed commands.

Don't forget to switch to the new stage using a new `FROM` instruction after the build is completed.

If you run into problems, try adding a single instruction at a time to the Dockerfile to ensure that you know what it soes before continuing to the next. Read all error messages carefully.

</details>

<details>

<summary>Hint2</summary>

A working example looks like this:

```Dockerfile
FROM node:alpine AS build-stage

COPY ./package.json  /app/package.json
COPY ./book.json  /app/book.json
COPY . /app
WORKDIR /app

RUN npm install
RUN npm run build

FROM nginx:alpine
COPY --from=build-stage /app/_book /usr/share/nginx/html
```

</details>

### Resources / Other things to try

* Look up the `node:alpine` image in more details. What if you wanted to use a specific node version? How could you let the node version be selected dynamically?
* [Advanced Multi-stage Build Patterns](https://medium.com/@tonistiigi/advanced-multi-stage-build-patterns-6f741b852fae) by Tõnis Tiigi is a really good writeup of the possibilities available when using multi-stage builds.

Don't stop yet! There are important security points in [Chapter 5](../5-security/README.md).