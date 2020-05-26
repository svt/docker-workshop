<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD024 -->
# Chapter 5: Security

In this chapter we will look at a few security caveats that are good to know about.

## Contents

1. [Keep your dependencies updated](#updated)
2. [Scrutinize your sources](#sources)
3. [Handle build secrets](#buildsecrets)
4. [Handle runtime secrets](#runtimesecrets)

---

## 5.1 Keep your dependencies updated <a name="updated"></a>

Congratulations, you are running your micro services in containers and have thus been granted wide-ranging control over your application's runtime.<br/>
In other words: Congratulations, you have been awarded with the trust to maintain the operating system (OS)-level dependencies of your applications.

Keeping your OS-level dependencies updated important because just like your library dependencies, your OS-level dependencies can contain security bugs that need to be patched.

Sounds scary? It doesn't have to be:

* The principle of maintaining OS-level dependencies are largely the same as for maintaining library dependencies
* You can rely on the work of the maintainers of your base image. You don't have to handle every package dependency separately, rather you work with readily-assembled collections of dependencies which are called base images, or distributions in more traditional Linux terms
* The most important thing to remember is to update them as regularly, maybe at the same time, as your library dependencies
* There are tools available to help you with the job

In this workshop section we will look at some approaches to handle updates of your base images.

### Different approaches for handling base image dependencies

As with library dependencies, you can go two orthogonal ways to achieve good dependency hygiene:

1. Depend on the most up-to-date version of your base image
1. Depend on a specific version of your base image

#### Practice: Depend on the most up-to-date version of your base image

How would you achieve this behaviour in your Dockerfile?

How does the approach behave? What are the advantages and disadvantages?

<details>
<summary>Hint</summary>

1. For depending on the most up-to-date version of your base image, you would specify a tag for your base image that is updated regularly, like `latest` in the example below:

```Dockerfile
FROM node:latest
```

Usually images have a `latest` tag, but there can be more continuously updated tags. Generally, Docker does not distinguish between mutable branches and immutable tags the way git does. Often it is clear from the name, but make sure to read up in your base image's documentation on the Docker Hub if a tag is continuously updated or not.

The image you build will be updated to the new version as soon as you build it again after the new version of the base image has been published.

Advantages:

* You will always get the latest fixes once you rebuild your image

Disadvantages:

* You don't get reproducible builds, which means that when you rebuild your image with based on the same git hash, it way not be the same as when you built it the last time. This can be problematic if changes or regressions are introduced upstream without you noticing

---

</details>

#### Practice: Depend on a specific version of your base image

How would you achieve this behaviour in your Dockerfile?

How does the approach behave? What are the advantages and disadvantages?

<details>
<summary>Hint</summary>

For depending on a specific version of your base image, use a tag of your base image that is not updated, usually represented by a tag named with a specific version number:

```Dockerfile
FROM node:13.10.1
```

Advantages:

* You have full control over your images and thus produce reproducible builds: The code from one git hash does always produce the same image. No nasty surprises.
* When you do big changes (major version updates) you can handle the risk with some extra testing.

Disadvantages:

* You have to have a process for explicitly updating base image versions regularly

---

</details>

#### Recommendations

##### If you maintain rapidly-developed services with good automated test coverage

Generally, **if you update your service regularly**, the approach of depending on the latest version might just work for you. In this case, the updates from your base image come in small increments, so the chances of you being caught off-guard by several regressions at the same time are relatively small. You are on top of the code of your service and if things break, you don't roll back, you just roll forward.

Also **automated integration tests help a lot with handling the risk** here.

**Watch out for changes in your product's life-cycle**: If you go from rapid development to more of a maintenance phase, you might need to re-evaluate your approach.

##### If you maintain mature services that need a way to minimize risk of upstream regressions

Depending on a specific version of your base image can work for you even if you don't change your code very often. The process of **updating dependencies is very clearly tracked in the code** and you can choose a good time for handling more risky updates to newer major versions.

Fundamentally, both approaches benefit very much from good automated integration test coverage. Depending on a specific version can however enable you to **handle the risk of upstream regressions with a more manual testing-based approach**.

The big challenge is to create a working update process and getting an answer to the question: When do I need to update?<br/> You can get help answering that question though: Integrate a security scanning tool, like Open Source tool [trivy](https://github.com/aquasecurity/trivy), into your [Continuous Delivery pipeline](https://github.com/aquasecurity/trivy#continuous-integration-ci).<br/>
Especially if you only do sporadic development on your service, consider running security scans on schedule with notifications to where you see them. That way you get a ping when a security vulnerability has been discovered that might affect your service. Depending on the severity of the vulnerability and the probability of exploitation in your service, you can make an informed decision about when to update.

##### Real life use-cases

The approaches described are consciously contrived. In reality you likely want to take an in-between approach.

Many base images have tags that are updated only within a major or minor version.

<details>
<summary>In our node example:</summary>

```Dockerfile
FROM node:13.10
```

or:

```Dockerfile
FROM node:13
```

---

</details>

In this way you can mix the two approaches: Continuous updates of patch/minor-versions but controlled updates for minor/major versions.

#### Choose long-term support releases

If you want **to minimize the maintenance burden, choose long-term support releases**. Use major versions that have a longer support cycle, if they are available. These major versions will get security and possibly bug fix updates for a longer time, while not otherwise changing the behaviour of the software.

#### Choose lightweight base images

Choosing more **lightweight base images can improve security** (like those built on the Linux distribution alpine). Since they contain fewer extras and optional packages, using them reduces the amount of exploitable security bugs.

Sophisticated images like `node` have many different permutations of tags: Not only for language runtime versions, but also for variants that build on different base images.

For example, when you specify, `FROM node:latest`, you get an image of the latest node version runtime based on the Linux distribution Debian in the version `stretch`. You can however also get a variant, the latest node based on alpine, by specifying `node:alpine`. The latest version within the major version 13 based on alpine would be `node:13-alpine`.

Check out the images' documentation on Docker Hub (t.ex.: [node](https://hub.docker.com/_/node)) to see what tags there are. These pages can be quite dense. As a tip: To see what a tag like `latest` is currently based on, look for it on the page and see what version tags are listed on the same row, these are usually equivalent.

---

## 5.2 Scrutinize your sources <a name="sources"></a>

As usual with software, you need to handle the risk of not knowing for sure if you can trust a source that you pull in software from.

In the case of Docker images, you can do a few things to minimize that risk:

* Always prefer [Docker Official Images](https://docs.docker.com/docker-hub/official_images/). These images are curated and evaluated by a Docker Inc. and get regular security updates. These images have the "official image" badge on the Docker Hub and lack a publisher name (i.e. just `node`, not `mytotallylegitcompany/node`).
* Prefer images built by the relevant upstream open source projects. They should have a reputation that can be lost when not doing a good job.
* Avoid images created by individuals.

It is not always easy to verify if an image was created by the upstream open source project. Here is a list of aspects you should look for to verify (more convincing ones first):

* Is the source code for the Dockerfile hosted in the same Github/Gitlab organization as the source code of the upstream open source project?
* Is the Docker image linked to from the official open source project's website or the README file?
* Have a look at the profile page of the image publisher (example for [selenium](https://hub.docker.com/u/selenium)). Does the information provided make sense? Things like wrong links to the project page or recent join dates should make you suspicious.

---

## 5.3 Handle build secrets <a name="buildsecrets"></a>

To build a Docker container, you often need to supply secrets to your language build tool, for example to download private library dependencies from our own build artifact repository.

These kinds of secrets are commonly called build secrets or build time secrets, since they are only needed at build time.

You need to be careful with these to not leak the secret by using it in the container.

### Practice: Retrieve a BUILD_ARG

Create a Dockerfile with the following content:

```Dockerfile
FROM nginx:latest

ARG BUILD_REPO_SECRET
RUN echo "Running a build using $BUILD_REPO_SECRET"
# Faking creation of build artifact
RUN touch build_artifact
```

Build the Docker image by running:

```
docker build --build-arg BUILD_REPO_SECRET=secr3t -t buildargtest .
```

Is there a way to retrieve the secret that was only needed for building in the finished container?

<details>
<summary>Hint</summary>

Yes you can! Just run:

```bash
docker image inspect buildargtest:latest | grep secr3t
```

Output:

```bash
                "BUILD_REPO_SECRET=secr3t",
```

So everyone who has access to this image also has access to the build secret.

You can of course also do this with images pulled from Docker Hub. In fact, `docker inspect` is a great way to learn about the images you want to use.

---

</details>

### Practice: Avoid leaking build arguments

To avoid the situation above, you can make use of something you already learned in Chapter 4: Multi-stage builds.

Try this:

1. Prevent leaking of the build secret by building in a build stage and copy the build artifacts into the final container.
1. Verify by running the `docker image inspect` command from above that the build secret is not leaked.

<details>
<summary>Hint</summary>

This Dockerfile does not leak build secrets:

```Dockerfile
FROM nginx:latest as build

ARG BUILD_REPO_SECRET
RUN echo "Running a build using $BUILD_REPO_SECRET"
# Faking creation of build artifact
RUN touch build_artifact

# Clean image without the BUILD_REPO_SECRET that we copy the build artifacts into
FROM nginx:latest

COPY --from=build build_artifact build_artifact
```

</details>

---

## 5.4 Handle runtime secrets <a name="runtimesecrets"></a>

Runtime secrets are secrets that are needed at the time a service actually runs. Examples are authorization credentials for accessing other networked services like databases.

According to [12factor](https://12factor.net/) etiquette, runtime secrets should always be [saved in the runtime environment](https://12factor.net/config). This enables us to use the same code in several environments, which in turn enables us to test code in a test environment before deploying it to production.

This also prevents leakage of secret information outside of runtime and build systems.

---

This concludes the current version of our docker workshop! We hope you enjoyed it and that you learned something. If you want to get in touch with us, please open an issue in the main repo at <https://github.com/svt/docker-workshop>. Thank's for playing!
