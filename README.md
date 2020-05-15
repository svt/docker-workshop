# Docker für alle

**A workshop in several chapters, originally held by the Developer Experience team at SVT.**

This workshop is for anyone who wishes to learn about docker and container management beyond copying and paste:ing from a guide somewhere on the internet. It is divided into multiple parts, but feel free to start at any point you feel is right for you. Some of the practical examples may depend on information presented earlier, in these cases we have tried to link back to the relevant information.

## Prerequisites

You need a personal computer and a working Docker installation, as outlined in the [prerequisites chapter](0-prereqs/README.md). This workshop is written primarily with Mac and Linux users in mind and tested on those systems, but most things *should* work the same on Windows.

## Where to start

If you are unsure about your skill level, start from [the very beginning](1-basics/README.md). It will not take you long to work through Chapter 1 if you have some previous experience, and you may still learn something new if you're lucky! We hope you enjoy your new docker skills.

Otherwise, if you feel very comfortable with the basics already, just dive in at any point that looks interesting in the overview.

### Chapter overview

* [Prerequisites](0-prereqs/README.md) - contains information and links that should get you a working docker installation.

1. [Basics](1-basics/README.md) - this chapter provides an overview of the basics of getting hold of docker images and running them locally. Start here if you have no previous experience at all.
2. [Dockerfiles](2-dockerfiles/README.md) - this chapter takes you through the process of writing Dockerfiles and building custom container images. Start here if you are already familiar with images and running containers but wish to understand more about how they are built and customized.
3. [Dynamic building](/3-dynamics/README.md) - an introduction to dynamically building and running images
4. [Multi-stage builds](/4-multi-stage-builds/README.md) - an overview of how to use multiple stages in your Dockerfile. Wow, you can build miniature pipelines now!
5. [Security](/5-security/README.md) - a guide to some common security risks inherent in using Docker, and how to mitigate them.

## Getting involved

We welcome contributions as well as questions, suggestions, reports of non-working examples, other feedback and success stories. Please feel free to open an issue in the main repo at <https://github.com/SVT/docker-workshop> to start the conversation. Please try to keep contributions:

* Accessible at the level specified
* Free from secondary requirements (more than basic knowledge of any particular programming language, for example)
* Tested on at least two platforms

----

## Open source licensing info

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />WorkshopS assets released under the <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>: [CC-BY-SA 4.0](LICENSE)

----

## Maintainers

- Frida Hjelm <https://github.com/svtfrida>
- Alexander Bethke <https://github.com/oolongbrothers>
- Johan Grimlund <https://github.com/grimlund>

## Credits and references

The DX:ers want to thank:
- Everyone who participated in the original workshop
- [Martin Flodin](https://github.com/mflodin) for suggesting an open source release of the material.