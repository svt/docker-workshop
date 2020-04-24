<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD024 -->
<!-- markdownlint-disable MD026 -->

# Chapter 3: Dynamics

In this chapter you will try building Docker images dynamically with ARGs and control them with ENV variables. You can use our prepared example in the `example-app` directory in this repository to try out the different methods.

## Contents

1. [Building with ARG and --build-arg](#arg)
2. [Controlling with ENV](#env)
3. [Combining ARG and ENV](#combining)
4. [Using the host filesystem](#volumes)

## 3.1 Building with `ARG` and `--build-arg` <a name="arg"></a>

The `ARG` instruction lets you define variables that exist in the scope of the docker build process. The resulting variable can have a default value defined and can also be overridden from the `docker build` command using the flag `--build-arg`. An example:

```Dockerfile
FROM ubuntu
ARG packages=curl
RUN apt-get update && apt-get install $packages
```

By default, this Dockerfile installs the `curl` package. But if we want to install something else instead, we can override the default when building the image using `--build-arg`:

```bash
docker build --build-arg packages=cowsay
```

The official documentation for the `ARG` instruction can be found [here](https://docs.docker.com/engine/reference/builder/#arg), we recommend that you have a look at it before continuing.

### Try: Use an ARG instruction to specify a directory to copy files from

Our example-app is a Gitbook containing info on a number of interesting things. We have pre-built it for you, so the subfolder `_book` contains static html content that you can copy and use in a container image. The example-app folder also contains a Dockerfile that starts a basic web server - the default nginx image, to be specific.

Use an `ARG` to let you specify a source directory for content to be copied to the web server image at build time. Then use `--build-arg` to put the static example site in your image, so that it is served when you start the container.

<details>

<summary>Hint1</summary>

If you don't know where to start, try modifying the Dockerfile to copy the files from a specific path first using `COPY`. Then consider how you can control the source path from the build command using an `ARG` instruction.

</details>

<details>

<summary>Hint2</summary>

You need to copy everything in the `_book` folder to the path `/usr/share/nginx/html/` in the image. One way to do this using `ARG` and `--build-arg` would be:

```Dockerfile
# Dockerfile
FROM nginx

ARG content_path

COPY $content_path /usr/share/nginx/html/
```

```bash
# Build command
docker build --build-arg content_path="_book" -t example-app:latest .
```

</details>

#### How you'll know it worked

When you start a new container based on the image you built, you should be able to browse the amazingly informative gitbook site.

---

## 3.2 Controlling with ENV <a name="env"></a>

Environment variables can be set in the Dockerfile using the `ENV` instruction (full documentation can be found [here](https://docs.docker.com/engine/reference/builder/#env)). These can have default values and/or be set on the command line in the `docker run` command (see the [complete command documentation](https://docs.docker.com/engine/reference/run/#env-environment-variables) for reference).

### Try: An example from the internet!

[This tutorial](https://medium.com/create-code/docker-environment-variables-and-nginx-93d7173f19ec) describes a scenario where we want to control the behaviour of our container depending on the value of and environment variable. Try it out!

*NB: The above example uses an environment variable called simply `ENV` to distinguish production and staging environments. Don't confuse the variable name with the instruction!*

---

## 3.3 Combining ARG with ENV <a name="combining"></a>

In most scenarios we use a combination of `ENV` and `ARG` instructions to acheive flexibility in Dockerfiles. For example, one common practice is to use an `ARG` to provide the value for an `ENV`. Read the ["Using ARG variables" section of the Dockerfile ARG documentation](https://docs.docker.com/engine/reference/builder/#using-arg-variables) to learn more about how they interact.

In short, an ARG will populate an ENV with the same name, but a value for the ENV explicitly set in the Dockerfile will override the value given by the ARG. ENV variables will be available to the processes in the resulting containers, while ARG variables may not be. [This blog post](https://dev.to/stevoperisic/dynamic-dockerfile-with-arg-2h62) describes the case very well.

**Please also have a look at the Secrets leakage section of chapter 5 for some examples of how ARG and ENV values can be exposed unintentionally.**

---

## 3.4 Using the host file system <a name="volumes"></a>

There is an option available when running containers that allow the container to interact with the local filesystem, so you don't have to copy everything to the image at build time. Common use cases are for example when we need persistent storage, such as when running a database or similar service.

**NB: This is not available on most microservice platforms, at least not in the default manner described here. Specifically it is not supported in SVT:s Molnet Apps. However, for local development purposes, it can be very useful.**

Persistent storage can be set up in multiple ways, but we'll keep it simple here. In order to make `/folder/on/docker/host/` available inside the container at the path `/path/inside/container/`, we specify this in the `docker run` command using the `-v` option:

```bash
docker run -v /folder/on/docker/host:/path/inside/container:ro image
```

The `:ro` part is optional and specifies that the access to the filesystem will be *read-only*. If it is omitted, the container will also be able to write to the path. There are other options available, you can find out more about them in the [documentation](https://docs.docker.com/engine/reference/run/#volume-shared-filesystems).

### Try: Serve content directly from the host filesystem

Remember the example in the beginning of this chapter? What if we were to just run the standard nginx image and inserted the `_book` directory at the appropriate path instead? Try it out.

<details>
<summary>Hint</summary>

The command would be something like:

```bash
docker run -v /home/myuser/docker-workshop/_book:/usr/share/nginx/html -p 80:8080 nginx
```

Note that the paths given in the command must be absolute.
</details>

Now for some nice building practices in [Chapter 4](../4-multi-stage-builds/README.md)!
