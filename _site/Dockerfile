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
