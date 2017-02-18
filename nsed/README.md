Nsed (No to Sed)
================

**Say No To Sed**

## Motivation

A while back i was given a task to deploy a bunch of software components onto an amazon EC2 instance:

- Graphite + Grafana
- Elasticsearch + Logstash + Kibana (ELK)
- Nagios
- ...
- ...

Immediately i thought to myself: "Hey, no sweat, im sure DockerHub has containers for all these guys!".

This means all i will have to do is write a *docker-compose* file that defines all of them. A quick lookup at the
DockerHub registery revieled that even though containers existed for most of these components, they were not exactly
what i needed. They were either hardcoding some configuration (for example elasticsearch port), or bringing in
extra components that i just did not need (for example StatsD with the graphite container).

This meant that i had to start writing my own docker files, and configure each component to my needs. And let me tell
you, there was a lot of configuration to do:

- Retention policies for graphite storage (carbon.conf)
- Elasticsearch port (elasticsearch.yml)
- Grafana datasources and security (config.js)
- Logstash filters (logstash.conf)
- Nagios checks (nagios.cfg)
- ...
- ...

Effectively, this meant that i had to manipulate a lot of different configuration files from within the Dockerfile.
So i did what every programmer would do:

1.  Bang my head against the table for a while.
2.  Freshen up my 'sed' skills.

After a couple of hours, and with very little hair left, i thought to myself: "Man, wouldn't it be great if every one
of these software components would provide a command line tool for configurating the damn thing?!"

Imagine that in order to configure the elasticsearch port, all you had to do was:

`$ elasticsearch config port=9201`

You may say, im a dreamer...but im not the only one...In fact, you have probably already seen something like this
in action: `$ git config user.email me@gmail.com`. Yeap, linus has done it again...

I then thought that there was nothing special about elasticsearch configuration. It is just a YAML file that can
be very easily manipulated using some basic python code (YAML can be deserialized into a dict), So why not write
a small python script that does exactly that? i could then use that script to configure any YAML file very easily:

`$ python yaml_patcher.py /etc/elasticsearch.yml port 9201`

The script also supprted nested configuration:

`$ python yaml_patcher.py /etc/elasticsearch.yml security.admin_password pass`

And even structured data:

`$ python yaml_patcher.py /etc/elasticsearch.yml security {"admin_password": "pass", "admin_user": "admin"} pass`

This proved to be extremely useful for me, especially because more and more tools are now switching to
YAML configuration. But i didn't want to stop there, i could do the same thing for many standrad file types:

- json
- init
- properties
- ...
- ...

And this is how *fileconfig* came to be, hope you enjoy it!