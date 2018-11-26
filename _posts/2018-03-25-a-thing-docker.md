---
layout: post
title: A thing or two about Docker
updated: 2018-03-25 17:37
---

## Deployment: A pain in itself

One of the biggest issues in software development is not creating it, but getting it to work everywhere. We've all heard of the phrase 'It works on my computer'. Keeping track of what settings were used in the dev environment is really challenging. Think about it, trying to remember what password you used for this database is a real pain, especially when you set up the database a while ago.

## The solution
One of the earlier solutions to this problem was taking a vm, installing your environment and then shipping it. Sounds good and even worked for a while. The problem here was that VMs are just too big. It took up a lot of space on hosting even if the storage was dynamically growing. Simply too big for a small application. Also getting multiple VMs to work together was a little difficult to the point that people lose interest in it. What the devs needed was a fast way to spin up a VM and get to work as fast as possible. Enter Docker. Docker is a containerization platform, i.e. it helps to build containers, which are light weight VMs

## Containers v/s VMs

A container is an abstraction that packages code and dependencies together, multiple containers can run on a single machine. Up until this point containers sound similar to VMs. This is true except that, the fundamental difference is that a container shares the kernel of the host OS, whereas a VM does not. As a result containers take up much less space as compared to VMs
A VM on the other hand is an abstraction of physical hardware. This creates a completely new hardware platform for each VM. Since the abstraction provided is of the hardware, a complete OS must be installed. This takes up a lot of space. This is where containers triumph over VMs.
![Containers](https://www.docker.com/sites/default/files/Container%402x.png)
![VMs](https://www.docker.com/sites/default/files/VM%402x.png)

## Docker Compose

Docker compose is a tool from docker for defining and running multi-container Docker applications. For example I have a website for showing me curated fine dining. Let's think about what all we need to create the website. First off a web server, then a database, maybe also an external session storage. For each of these entities I can create a container and then tie them together. Docker compose allows me to define and tie containers together in a docker compose file which is of YAML format. Using compose takes 3 steps:
1. Create a Dockerfile to create the environment of your choice(Usually this step is rarely done as a lot of pre-made images already exist)
2. docker-compose.yml file for defining all the services which your application requires
3. docker-compose up command to create it.
That's how simple it is.

Here's an example of a Dockerfile
{% highlight plaintext %}
#
# Simple example of a Dockerfile
#
FROM ubuntu:latest
MAINTAINER Dylan Dsouza "dsouzadyn@gmail.com"

RUN apt-get update
RUN apt-get install -y python python-pip wget
RUN pip install Flask

ADD hello.py /home/hello.py

WORKDIR /home
{% endhighlight %}

And here is an example of a docker-compose.yml file.

{% highlight yaml %}
version: '2'

services:
  web:
    build: .
    ports:
     - "5000:5000"
    volumes:
     - .:/code
  redis:
    image: redis
{% endhighlight %}

## The situation today

Containers are the future. A lot of companies have adopted this method of deployment as it is much easier and give's the developer full control over his/her application. One major factor is the reduction of hosting costs. Containers make the whole dev-ops business a whole lot transparent and the developer community likes that. Containers are here to stay, until another ground breaking technology comes along. It's safe to say that won't happen for quite some time.

### Refences
[https://www.docker.com/what-docker](https://www.docker.com/what-docker)
[https://docs.docker.com/compose/](https://docs.docker.com/compose/)
[https://www.docker.com/what-container](https://www.docker.com/what-container)
All images taken from [Docker](https://www.docker.com)
