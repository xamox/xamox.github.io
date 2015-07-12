---
layout: post
title:  "Docker & Jekyll"
date:   2015-07-12 10:00:00
---
![Image docker-logo](/images/docker.png) ![Image jekyll-logo](/images/jekyll.png)

When I recently decided to build a new site it was a no brainer for me to do it using [Docker](http://docker.com) and [Jekyll](http://jekyllrb.com/); both are setup well for a statically built, immutable environment.  Docker has gotten a lot of press recently for being a tool for "containerized environments" that is revolutionizing the devops, deployment, and testing.  I had heard of Docker and CoreOS a few years back, especially since container support has been in the mainline linux kernel since 2008 I believe, but didn't actually start using it until about 9 months ago.  I now use it daily and I definitely believe it is the direction everything is moving. I'm in the camp for microservices + containers.  I've been building and shipping code so long that I knew it made things way easier after just playing around with it for a bit. The first thing I used it for was build a [SimpleCV container](https://registry.hub.docker.com/u/sightmachine/simplecv/) that way I can gurantee that if someone is using it on their system it is exactly the same.  It makes debugging the "works on my machine" so much easier, and now it is as easy as running a single command to git the software and another to run it.

    # Download and Run SimpleCV using Docker
    > sudo docker pull sightmachine/simplecv
    > sudo docker run -it --rm --name="simplecv" sightmachine/simplecv


But I digress.  Given that this doesn't require much I will just show you the code and give you the run down.
The first thing you will need to do is ensure you have [Docker installed](https://docs.docker.com/installation/) as well as [docker compose installed](https://docs.docker.com/compose/install/).  At the time of writing this, I was using Docker 1.6.2.

Here is the Dockerfile:

    # Save this file as:
    # /path/to/jekyll/Dockerfile
    #
    # To build:
    # sudo docker build -t="website" .
    #
    # To run:
    # sudo docker run --name="website" website jekyll -H 0.0.0.0 serve
    # 
    FROM ubuntu:14.04

    RUN apt-get update && apt-get install -y ruby-full nodejs make
    RUN gem install jekyll

    VOLUME ["/root/website"]
    EXPOSE 4000
    COPY . /root/website
    WORKDIR /root/website
    RUN jekyll build


This file would go into the same directory that your Jekyll base directory would be. That way it makes it easy to live in git, clone elsewhere and build. Automate the build when push via git, etc.  You do not necessarily have to even run the build portion above as there is also another useful tool that you should have installed earlier called docker-compose.  This tool makes it easy to spin containers up and down in a local environment as well as link them to each other or volumes for data storage.


Here is the docker compose file to spin up the container and mount the directory on the host side in the container so that we may develop on it.

    # Save this file as:
    # /path/to/jekyll/docker-compose.yml
    website:
        build: .
        command: jekyll serve -H 0.0.0.0
        restart: on-failure
        ports:
            - "4000:4000"
        volumes:
            - .:/root/website #This overrides the static folder that lives in container for development

If you say wanted to run this in production then you could build it without the "volumes" section and it would be copied into the contain upon building.
Now that we have our dockerfile and compose file we just need to run it:

    > cd /path/to/jekyll
    > sudo docker-compose up
    Recreating portfolio_website_1...
    Attaching to portfolio_website_1
    website_1 | Configuration file: /root/website/_config.yml
    website_1 |             Source: /root/website
    website_1 |        Destination: /root/website/_site
    website_1 |       Generating... 
    website_1 |                     done.
    website_1 |  Auto-regeneration: enabled for '/root/website'
    website_1 | Configuration file: /root/website/_config.yml
    website_1 |     Server address: http://0.0.0.0:4000/
    website_1 |   Server running... press ctrl-c to stop.

Now you should be able to go to [http://localhost:4000](http://localhost:4000) in your browser and see the site is built and running.


This has made it extremely easy to recreate my environment and test in multiple locations as long as I have docker installed.  The best part is I don't even need to have jekyll or any of the associated libraries installed to use the software, they all come bundled in the container.   I didn't get into too much detail on how to actually build a website using jekyll or build a blog.  That is too much for a single post.  But you can see how easy from the above that you could get an environment setup to at least start playing with jekyll.  I highly recommend going through the getting started section of [docker docs](https://docs.docker.com/).  In future post I will expand on using docker with other tools and it will begin to make sense why it's a useful tool.

