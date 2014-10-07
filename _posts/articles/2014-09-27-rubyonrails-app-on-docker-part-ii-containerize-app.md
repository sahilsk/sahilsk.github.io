---
layout: post
title: "RubyOnRails App on Docker: Part-II Containerizing App"
modified: null
categories: articles
excerpt: null
tags: 
  - RoR
  - Docker
  - Deploy
  - 12factor.net
comments: true
share: true
image: 
  feature: null
date: 2014-10-1T15:12:48+05:30
author: sahilsk
published: true
---

RubyOnRails App On Docker: Part-II How are we doing?
===============

Assumption:

You're free to name your application and docker namespace anything you want. However, to make this article more readable, i'm using undersigned names.

	application name: `dailyReport`
	docker user namespace:  `myDockerfiles`


Index
 
- Setup and Install database
- Containerize RoR Apps
- Setup Reverse Proxy using Nginx



1. Setup and Install database
-------------------------

	For this application we'll use MySQL. There are two ways to run MySQL
	- Run without container
	- Run inside Docker


	Q. Any performance impact when runng inside Container?
	---

	In docker, cpu performance is native, disk latency is native,
	memory is not native but could be made. Same is true for network latency.

	Nowadays hardware are cheap but software are costly. So, we don't need to worry about little memory that Docker keep aside. If you really want to squash every single drop, then there are ways to do so. Network latency also can be made as fast as of native. This small minor reduction we can bear.

	So, having compromised with Memory and Network latency, we're proceeding to Dockerize MySQL instance.

	Luckily, there is already mysql Dockerfile ready in the docker hub: [MySQL Dockerfile](https://github.com/dockerfile/mysql) 


**dockerfile/mysql**
{% highlight bash linenos %}
#
# MySQL Dockerfile
#
# https://github.com/dockerfile/mysql
#

# Pull base image.
FROM dockerfile/ubuntu

# Install MySQL.
RUN \
  apt-get update && \
  DEBIAN_FRONTEND=noninteractive apt-get install -y mysql-server && \
  rm -rf /var/lib/apt/lists/* && \
  sed -i 's/^\(bind-address\s.*\)/# \1/' /etc/mysql/my.cnf && \
  sed -i 's/^\(log_error\s.*\)/# \1/' /etc/mysql/my.cnf && \
  echo "mysqld_safe &" > /tmp/config && \
  echo "mysqladmin --silent --wait=30 ping || exit 1" >> /tmp/config && \
  echo "mysql -e 'GRANT ALL PRIVILEGES ON *.* TO \"root\"@\"%\" WITH GRANT OPTION;'" >> /tmp/config && \
  bash /tmp/config && \
  rm -f /tmp/config

# Define mountable directories.
VOLUME ["/etc/mysql", "/var/lib/mysql"]

# Define working directory.
WORKDIR /data

# Define default command.
CMD ["mysqld_safe"]

# Expose ports.
EXPOSE 3306
{% endhighlight %}


Dockerfile is quite simple. Isn't it? In starting few lines, we're installing mysql server and granting user root all privileges.

Line 24, however, need little elboration. Data directory will enable direct access to configuration and data files. This I'll answer in my part-III. For now,  lets put parts rolling.

Let's setup and run mysql

{% highlight bash %}
#Pull and Run mysql image
sudo docker run -d --name mysql -p 3306:3306 dockerfile/mysql
{% endhighlight %}


This one line is suffice to run mysql server up and running.
To verify , we'll start mysql client using the same image but different command.


{% highlight bash %}
	sudo docker run -it --rm --link mysql:mysql dockerfile/mysql bash -c 'mysql -h $MYSQL_PORT_3306_TCP_ADDR'
{% endhighlight %}

### How to connect our application with database?

There are actually two way to specify mysql connection to our app.


- mouting default mysql unix sock

	This is useful if you don't want to run mysql publicly. And for this tutorial, i've done the same.
	I had database installed on my server. So, I'll mount `/var/run/mysqld` on my container, thereby enabling rails to find default mysql endpoint to connect with.

		docker run -d   -p 49172:80  -v /var/run/mysqld:/var/run/mysqld:ro  --restart="always" -e "RAILS_ENV=production"  myDockerfiles/dailyreport

- Specify connection string

	If you want to use database running somewhere accessible through IP/Port, then you can specify these connection string in an environment variable `DATABASE_URL`

		docker run -d   -p 49172:80  --restart="always" -e "RAILS_ENV=production"  -e "DATABASE_URL='mysql2://username:password@IP/DB_NAME"  myDockerfiles/dailyreport





2. Containerize RoR Apps
-------------------------

	RoR framework already comes with sensible default best practices of Software Development.
	However, there are few configuration that i'd like to stress on:

	- Session Storage

		Store session information in database. This will enable our app to behave more like stateless app. Also, this is essential if we want to scale our infrastructure further 

	- Secrets

		Database configuration, RoR Secret key (SECRET_KEY_BASE) , environment , smtp credentials, or other 3rd party addons secret that your app might be using, should not be hardcoded in configuration file. Instead, they should be picked from environment.


	Here's one example showing database credentials being picked from environment.

__config/database.yml__
{% highlight yaml %}
production:
  <<: *default
  database: <%= ENV['DATABASE_PROD'] %>
  username: <%= ENV['DATABASE_USERNAME'] %>
  password: <%= ENV['DATABASE_PASSWORD'] %>
{% endhighlight %}



Similary, we'll specify SMTP parameters. If you're using any 3rd party service, like (mailgun, aws secrets etc), credentails should not be hardcoded, rather should be set in environment for the process to pick while running.

For setting up database credentials in environment, there's a shorthand provided by RoR:

	 DATABASE_URL="mysql2://myuser:mypass@localhost/somedatabase"

NOTE: Setting `DATABASE_URL` environment variable will take precendence over config file params, & merge with config files to populate db connection setting. 

### App Server for RoR: Unicorn

We'll choose widely adopted Unicorn as our web server.

**unicorn.rb**
{% highlight ruby linenos %}
# Set the working application directory
# working_directory "/path/to/your/app"
working_directory "/opt/dailyReport"
 
# Unicorn PID file location
pid "/var/run/unicorn.pid"
 
# Path to logs
stderr_path "/var/log/dailyReport/unicorn.err.log"
stdout_path "/var/log/dailyReport/unicorn.log"
 
# Unicorn socket
listen "/tmp/unicorn.dailyReport.sock"
 
# Number of processes
## Rule of thum: 2x per core
worker_processes 2
 
# Time-out
timeout 30
{% endhighlight  %}


### Web Server for RoR: Nginx

As the rail guides says, best practice to  serve assets is through nginx. Here nginx will also serve as reverse proxy by masking unix socket and giving illusion of app running at http port.

To make assets serving faster, we'll gzip our styelsheets and javascripts. How? This is not in the scope of this article, however in rails simple `rake assets:precompile`  command does the trick.
Our web server will have assets block that will server these compressed files.

{% highlight bash %}
upstream app {
    # Path to Unicorn SOCK file, as defined previously
    server unix:/tmp/unicorn.dailyReport.sock fail_timeout=0;
}

server {
    listen 80;
    server_name localhost;

    # Application root, as defined previously
    root /opt/dailyReport/public;

    try_files $uri/index.html $uri @app;

    location @app {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_pass http://app;  # point to our upstream server list
    }
	
	#Server compressed assets
	location ~ ^/(assets)/  {
	  gzip_static on; # to serve pre-gzipped version
	  expires max;
	  add_header Cache-Control public;
	}
	
    error_page 500 502 503 504 /500.html;
    client_max_body_size 4G;
    keepalive_timeout 10;
} 
{% endhighlight %}



#### How will we start our app?

Here comes run.sh into picture. We've written our startup script in this file.

**run.sh**
{% highlight bash %}
RAILS_ENV=$RAILS_ENV
: ${RAILS_ENV:="development"}
export RAILS_ENV

SECRET_KEY_BASE=$SECRET_KEY_BASE
: ${SECRET_KEY_BASE:="f38c575fcf0a2b0e7c7f002a873d54d78104581ebe069bf2b1afad04014d1e10245b259b872b0e12189ef2ce3fca4c73a9b5103aaf4aad1f4"}
export SECRET_KEY_BASE=$SECRET_KEY_BASE

## Setting DB
DB_NAME="dailyReport_${RAILS_ENV}"
#DATABASE_URL="mysql2://root:root@localhost/${DB_NAME}"

# Trap sigkill and sigterm: otherwise dockr stop/start will complain for stale unicorn pid
trap "pkill unicorn_rails ; exit " SIGINT SIGTERM SIGKILL

echo "Stopping  unicorn_rails, if already running"
pkill unicorn_rails

echo "cleaning tmp files"
rm -rf tmp/*

echo "Restart Reverse Proxy"
service nginx restart

echo "Running unicorn"
bundle exec unicorn_rails -c /etc/dailyReport/unicorn.rb -E $RAILS_ENV -d

{% endhighlight %}

let's wrap these lines inside `run.sh` file which will serve as our app startup script.

We wrote `run.sh`,and `unicorn.rb` file. We've also replaced hardcoded database, SMTP and 3rd party credentials with environment variables.
Now now need to wrap our app and unicorn in a container.


For this we'll create two dockerfiles

- Base Dockerfile 

	It'll contain latest version of ruby and rails installed.
	Since ruby 1.9.x EOL is near. The newer 2.1.x version is comparatively fast, and bug free.
	So, we'll use ruby 2.1.2 and we'll get it installed by rbenv. It'll also help us to update ruby version without re-building docker image again. __how?__



{% highlight bash %}
# Note down its container id
docker run -it baseDockerImage /bin/bash
	$ rbenv install ruby 2.1.3
	$ exit
docker commit -m "ruby2.1.3" CONTAINER_ID  
{% endhighlight bash %}		

- Main Dockerfile
	
	This will be our Dockerfile for our application. It'll include 'unicorn.rb', 'run.rb' and 'reverse proxy' configuration. Basically, everything that's required to run ror app natively.


------

**myDockerfiles/base_ruby**

{% highlight bash %}
#
# Ruby with rbenv Dockerfile
#

# Pull base image.
FROM dockerfile/ubuntu	

# Install some dependencies
RUN apt-get update
RUN apt-get install -y git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties

# Install rbenv to install ruby
RUN git clone git://github.com/sstephenson/rbenv.git /usr/local/rbenv
RUN echo '# rbenv setup' > /etc/profile.d/rbenv.sh
RUN echo 'export RBENV_ROOT=/usr/local/rbenv' >> /etc/profile.d/rbenv.sh
RUN echo 'export PATH="$RBENV_ROOT/bin:$PATH"' >> /etc/profile.d/rbenv.sh
RUN echo 'eval "$(rbenv init -)"' >> /etc/profile.d/rbenv.sh
RUN chmod +x /etc/profile.d/rbenv.sh

# Install rbenv plugin: ruby-build
RUN mkdir /usr/local/rbenv/plugins
RUN git clone https://github.com/sstephenson/ruby-build.git /usr/local/rbenv/plugins/ruby-build

# Let's not copy gem package documentation
RUN echo "gem: --no-ri --no-rdoc" > ~/.gemrc

ENV RBENV_ROOT /usr/local/rbenv
ENV PATH $RBENV_ROOT/bin:$RBENV_ROOT/shims:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# Install ruby
RUN rbenv install 2.1.2
RUN rbenv local 2.1.2
RUN rbenv global 2.1.2



## Install Rails
RUN apt-get install -y software-properties-common
RUN add-apt-repository ppa:chris-lea/node.js
RUN apt-get update
RUN apt-get install -y nodejs

## Finally, install Rails
RUN gem install rails
RUN rbenv rehash

CMD /bin/bash
{% endhighlight %}


	Let's build and tag it

{%  highlight bash %}
docker build -t "myDockerfiles/base_ruby" .
# Run and test
docker run -it --rm  myDockerfiles/baseRubyImg /bin/bash -c 'ruby -v'
{% endhighlight %}
	
Here comes our main app Dockerfile that we will use 


**myDockerfiles/main**
{% highlight bash linenos %}
#
# dailyReport in Container
#
 
# Pull base image.
FROM  myDockerfiles/base_ruby
 

# Fill dependencies for mysql2 gem
RUN apt-get install -y libmysqlclient-dev libmysqlclient18 ruby-dev
 
# Install Nginx.
RUN \
  add-apt-repository -y ppa:nginx/stable && \
  apt-get update && \
  apt-get install -y nginx && \
  rm -rf /var/lib/apt/lists/* && \
  chown -R www-data:www-data /var/lib/nginx

# Pull repository from private github repos
 
### Create .ssh dir in home directory
RUN mkdir -p /root/.ssh
# Add your private key here. (Create a separate key, so that you can revoke it later)
ADD ./id_rsa /root/.ssh/id_rsa
RUN chmod 700 /root/.ssh/id_rsa
RUN echo "Host github.com\n\tStrictHostKeyChecking no\n" >> /root/.ssh/config
 

# Setup Reverse Proxy : Add reverse proxy config here
ADD ./dailyReport_nginx.conf /etc/nginx/sites-enabled/default
RUN service nginx reload && service nginx restart

 
WORKDIR /opt/dailyReport

# Pull project : Replace with your github handle and repository
RUN git clone git@github.com:sahilsk/dailyReport.git .
 
# Install gem
RUN gem install bundler
RUN bundle install
RUN rbenv rehash

# Pre-compile app production assets
RUN RAILS_ENV=production bundle exec rake assets:precompile
 
# Add unicorn config here 
ADD ./unicorn.rb /etc/dailyReport/unicorn.rb

# Run script
ADD ./run.sh /etc/dailyReport/run.sh


# Define mountable directories.
VOLUME ["/etc/dailyReport", "/var/log/dailyReport", "/etc/nginx/sites-enabled", "/etc/nginx/certs", "/etc/nginx/conf.d", "/var/log/nginx"] 

# Expost port 80
EXPOSE 80

# Set environment variables
ENV RAILS_ENV development

#
CMD /bin/bash /etc/dailyReport/run.sh
{% endhighlight %}


Let's build out main app now.


	$  docker build -t myDockerfiles/dailyreport .

It all goes well, you now have two images built succesfully.



Having build two images, you can see them using docker commands

	$ docker images


Now lets run our app container

	$docker run -d -p 49172:80  -v /var/run/mysqld:/var/run/mysqld:ro  --restart="always" -e "RAILS_ENV=production"  myDockerfiles/dailyreport

You can visit localhost:49172 and confirm if your app is launched 

You can run more than one instance. Simple change the port and execute.
	
	$docker run -d -p 49173:80  -v /var/run/mysqld:/var/run/mysqld:ro  --restart="always" -e "RAILS_ENV=production"  myDockerfiles/dailyreport


References:
==========

- Dockerfile and scripts used here are on [github](https://github.com/sahilsk/RoR-Dockerized)