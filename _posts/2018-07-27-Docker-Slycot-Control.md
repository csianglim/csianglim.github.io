---
layout: blog
title: "Dockerfile, Slycot and Control"
date: "2018-07-27"
author: ["Siang Lim"]
---

# Containerizing Slycot and the Python Control Systems Library
In this post, I'll show you how to containerize a Python application that uses Slycot and the [Python Control](https://github.com/python-control/python-control) library. Installing Slycot is tricky because you'll need a FORTRAN compiler and a bunch of other dependencies, which isn't included in most off-the-shelf Docker images like [alpine-python](https://github.com/jfloff/alpine-python).

# Docker introduction
Docker containers are built from base images. A base image can be an operating system image like Ubuntu, Debian or CentOS. We can have intermediate images called **layers** on top of our base image. Every line of instruction in our Dockerfile creates a new layer. These layers make up the final Docker container.

Here's a visual of what it looks like:

![docker diagram]({{ "/assets/images/container_layers.png" | absolute_url }}){:class="center-image"}

*Diagram from the [Docker Blog](https://blog.docker.com/2015/10/docker-basics-webinar-qa/).*

# Creating a Dockerfile
One common challenge with Docker containers is keeping the image size small. An Ubuntu base image can be over 600MB. This can easily balloon to over 1-2GB once we start installing more layers.

It would be a good practice to start with a minimal base image like [Alpine](https://github.com/gliderlabs/docker-alpine) and just add the packages we need.

Here's the Dockerfile, we'll start with a Python 3.6 image, and add the dependencies needed by Slycot, numpy and scipy:

```
FROM python:3.6-alpine

RUN apk --update add git openssh && \
    rm -rf /var/lib/apt/lists/* && \
    rm /var/cache/apk/*

RUN apk --no-cache --update-cache add \
	gcc \
	gfortran \
	g++ \
	build-base \
	wget \
	freetype-dev \
	libpng-dev \
	openblas-dev

RUN ln -s /usr/include/locale.h /usr/include/xlocale.h
```

To prevent Docker from unnecessarily rebuilding pip packages, we'll use this little [trick](https://www.aptible.com/documentation/enclave/tutorials/faq/dockerfile-caching/pip-dockerfile-caching.html) of adding `requirements.txt` to our app directory before doing `pip install`:

```
ADD requirements.txt /app/
WORKDIR /app

RUN pip install --no-cache-dir \
	numpy \
	slycot \
	scipy \
	git+https://github.com/python-control/python-control
```

Side note: I was having some trouble installing the control library properly using `pip install control`, that's why I'm using the Github link. When I tried the pip package in December 2017, it had some [compatibility issues](https://github.com/python-control/python-control/pull/170) with the latest version of Scipy. The issue seems to be fixed in the latest control 0.8.0 version (as of July 2018).

If your app has any addition requirements you can put it in `requirements.txt`. Here's the last part of the Dockerfile:

```
RUN pip install -r requirements.txt
ADD . /app

ENTRYPOINT ["python"]
CMD ["app.py"]
```

# Docker Hub
This Docker container is also available in this [GitHub repo](https://github.com/csianglim/alpine-slycot-control) or this [DockerHub repo](https://hub.docker.com/r/csianglim/alpine-slycot-control/).

Alternatively, use Docker pull:

```
docker pull csianglim/alpine-slycot-control
```

then run it:

```
docker run csianglim/alpine-slycot-control
```


# Other known issues
FYI Slycot seems to be really fussy about its dependencies, especially numpy versions. I tried using [abn/scipy-docker-alpine](https://github.com/abn/scipy-docker-alpine) as my base image to avoid compiling numpy, scipy and speed up my Docker builds, but Slycot didn't like it. So if you're having trouble, try the latest numpy and scipy version.

# Conclusion
The `control` library requires scipy and numpy, which are big dependencies, making the final Docker image over 700MB. However, it's very likely we could optimize the Dockerfile further and shrink the container size. A follow-up article will be posted when I figure that out.
[]: 
