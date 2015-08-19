---
layout: post
title:  "ELK in a Can!"
date:   2015-08-18 10:00:00
---
![Image canned-elk](/images/canned-elk.jpg)

For the past few months I've been playing with another useful tool besides <a href="http://docker.com">docker</a> that I find extremely useful for many types of not just devops applications, but all kinds of use cases where you may want to monitor some type of machine.   The tool, or stack for that matter that I'm talking about is typically refered to as the <a href="https://www.elastic.co/">ELK stack</a>.  This is Elasticsearch, Logstash, and Kibana.  Each tool in the stack performs a different task but together they are a very lethal combo that I think any developer that has any grit should at least investigate.   ELK could be used as a tool for data collection for the Internet of things devices, to monitor servers, machines around the globe.  It is very fault tolerant if configured correctly and is designed to scale.  There are some other interesting properties for elasticsearch in the way it does reverse indexes to store and retrieve data, and the way many parts of the stack can be swapped out or configured to handle many various streams of data.

Here is an example screenshot of what the tool looks like as an end user:
![Image screenshot-kibana](/images/screenshot-kibana.jpg)


But enough gabbing about how great it is, time to get on with how to use it. Now the reason I'm writing this post as when I started to investigate ELK almost a year ago there wasn't a very quick way to get the system up and running for testing.  This was mainly due to the fact that each part of the chain in the stack uses different programming languages and tools to run.  Logstash is built on ruby, logstash forwarder (a.k.a. lumberjack) built in go, elasticsearch in java, and kibana in many various other tools and languages.  After a while of playing around I was able to get something setup to use another one of my favorite tools which is docker.


Before we get going we need to understand what exactly is going on behind the scenes in the ELK stack.   In short it's a system that you pass in log infomation into.  This can be done in a variety of ways, JSON, log parsers, and many other tools.  The input to the stack is logstash, hence logs going in.  Logstash is way more in depth than that, and I suggest reading <a href="http://www.amazon.com/Logstash-Book-James-Turnbull-ebook/dp/B00B9JQTCO/">the logstash book</a> for a good quick read on how to get it configured in many ways.  It supports many various <a href="https://www.elastic.co/guide/en/logstash/current/input-plugins.html">input plugins, as well as <a href="https://www.elastic.co/guide/en/logstash/current/output-plugins.html">output plugins</a> (one of them obivously being elasticsearch).  It also includes many filters that allow you to either transform the data or declare how you will index it as it's ingested into the system.  What's nice is that many logstash servers can feed into a single elasticsearch server, or elasticsearch cluster for that matter (scaling).  For this example we are just going to use the generator input plugin to fake data just to get a test system up and running from which could be expanded upon.

The configuration for the logstash input portion of the plugin to use the generator is as follows:

    input {
      generator {
        lines => [
            "This",
            "data",
            "is",
            "generated"
        ]
        threads => 8
        tags => ['fake', 'data']
        type => 'canned_elk'
      }
    }

Now don't worry all this will work with a single simple command, ala, "docker-compose up" with the code, but this is to get an idea of how easy the configuration is. The output section will look as follows:

    output {
      elasticsearch { host => 'elasticsearch' }
      stdout { codec => rubydebug }
    }

Notice in this section the host is labeled "elasticsearch", which seems strange because it requires and IP address or domain name.  In this case the domain name is being created as part of the docker container building and linking.  That hostname is available through the links as defined in the docker compose file:

    logstash:
      build: logstash-contrib
      volumes:
          - logstash-contrib:/config/
      command: logstash -f /config/logstash.conf
      ports:
          - "25826:25826"
      links:
          - elasticsearch
    kibana:
      build: kibana
      links:
          - elasticsearch
      ports:
          - "5601:5601"
    elasticsearch:
        image: elasticsearch

This compose file will essentially pull down the base containers you need if you don't have them, it will also build logstash with the plugins as well.  It's that easy, you just run the following and get similiar output.

    
    > docker-compose up #may need to run with sudo
    # ...
    # downloading and building base containers stuff here
    # ...
    Successfully built 013931dd37f6
    Attaching to cannedelk_elasticsearch_1, cannedelk_logstash_1, cannedelk_kibana_1
    elasticsearch_1 | [2015-08-19 03:12:34,171][INFO ][node                     ] [Amazon] version[1.7.1], pid[1], build[b88f43f/2015-07-29T09:54:16Z]
    elasticsearch_1 | [2015-08-19 03:12:34,172][INFO ][node                     ] [Amazon] initializing ...
    elasticsearch_1 | [2015-08-19 03:12:34,247][INFO ][plugins                  ] [Amazon] loaded [], sites []
    elasticsearch_1 | [2015-08-19 03:12:34,278][INFO ][env                      ] [Amazon] using [1] data paths, mounts [[/usr/share/elasticsearch/data (/dev/disk/by-uuid/3d4e1e9d-fc31-4306-a5f6-6b4aefcded85)]], net usable_space [93.7gb], net total_space [226.3gb], types [ext4]
    elasticsearch_1 | [2015-08-19 03:12:36,663][INFO ][node                     ] [Amazon] initialized
    elasticsearch_1 | [2015-08-19 03:12:36,663][INFO ][node                     ] [Amazon] starting ...
    elasticsearch_1 | [2015-08-19 03:12:36,733][INFO ][transport                ] [Amazon] bound_address {inet[/0:0:0:0:0:0:0:0:9300]}, publish_address {inet[/172.17.0.13:9300]}
    elasticsearch_1 | [2015-08-19 03:12:36,756][INFO ][discovery                ] [Amazon] elasticsearch/IPCm0fP7RDiR59AAGqaqrA
    elasticsearch_1 | [2015-08-19 03:12:40,524][INFO ][cluster.service          ] [Amazon] new_master [Amazon][IPCm0fP7RDiR59AAGqaqrA][b5cb60ca470a][inet[/172.17.0.13:9300]], reason: zen-disco-join (elected_as_master)
    elasticsearch_1 | [2015-08-19 03:12:40,542][INFO ][http                     ] [Amazon] bound_address {inet[/0:0:0:0:0:0:0:0:9200]}, publish_address {inet[/172.17.0.13:9200]}
    elasticsearch_1 | [2015-08-19 03:12:40,542][INFO ][node                     ] [Amazon] started
    elasticsearch_1 | [2015-08-19 03:12:40,561][INFO ][gateway                  ] [Amazon] recovered [0] indices into cluster_state
    kibana_1        | {"@timestamp":"2015-08-19T03:12:59.703Z","level":"info","message":"Listening on 0.0.0.0:5601","node_env":"production"}
    elasticsearch_1 | [2015-08-19 03:13:02,519][INFO ][cluster.service          ] [Amazon] added {[logstash-3fac7664e84f-1-2010][c7Nat_5UR2umX-TcJI88LQ][3fac7664e84f][inet[/172.17.0.16:9300]]{client=true, data=false},}, reason: zen-disco-receive(join from node[[logstash-3fac7664e84f-1-2010][c7Nat_5UR2umX-TcJI88LQ][3fac7664e84f][inet[/172.17.0.16:9300]]{client=true, data=false}])
    ...


This will continue to run until you hit ctrl+c.  

!!! WARNING !!!: This may hammer on your system as it will be generating and indexing a lot of data really fast.

Once it is up and running you will just need to go to <a href="http://localhost:5601">http://localhost:5601</a> in your browser. This should load up the kibana interface and you should see something similiar to the kibana image shown before.  You will also need to tell it what to index on so you will need to select @timestamp from the dropdown menu and click the "create" button.  Then if you click the "discover" tab you should start seeing graphs come through.

As you can see we basically have ELK stack in a can.  Hopefully it has been helpful.  Keep in mind now that all of this has been containerized it would be very easy to roll out the various containers and link them together using something like ansible or the up and coming docker machine.

You can get all the code at the repo here:
<a href="https://github.com/xamox/canned-elk">https://github.com/xamox/canned-elk</a>