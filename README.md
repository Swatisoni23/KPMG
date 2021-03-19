# 3 tier App  

### Step 1: Build and run your app with Compose  

1. From your project directory, start up your application by running docker-compose up.  

```
$ docker-compose up
Creating network "composetest_default" with the default driver
Creating composetest_web_1 ...
Creating composetest_redis_1 ...
Creating composetest_web_1
Creating composetest_redis_1 ... done
Attaching to composetest_web_1, composetest_redis_1
web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
redis_1  | 1:C 17 Aug 22:11:10.480 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis_1  | 1:C 17 Aug 22:11:10.480 # Redis version=4.0.1, bits=64, commit=00000000, modified=0, pid=1, just started
redis_1  | 1:C 17 Aug 22:11:10.480 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
web_1    |  * Restarting with stat
redis_1  | 1:M 17 Aug 22:11:10.483 * Running mode=standalone, port=6379.
redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
web_1    |  * Debugger is active!
redis_1  | 1:M 17 Aug 22:11:10.483 # Server initialized
redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
web_1    |  * Debugger PIN: 330-787-903
redis_1  | 1:M 17 Aug 22:11:10.483 * Ready to accept connections
```

Compose pulls a Redis image, builds an image for your code, and start the services you defined. In this case, the code is statically copied into the image at build time.  

2. Enter http://0.0.0.0:5000/ in a browser to see the application running.  

If you’re using Docker natively on Linux, Docker for Mac, or Docker for Windows, then the web app should now be listening on port 5000 on your Docker daemon host. Point your web browser to http://localhost:5000 to find the Hello World message. If this doesn’t resolve, you can also try http://0.0.0.0:5000.  

If you’re using Docker Machine on a Mac or Windows, use docker-machine ip MACHINE_VM to get the IP address of your Docker host. Then, open http://MACHINE_VM_IP:5000 in a browser.  

You should see a message in your browser saying:  
```
Hello World! I have been seen 1 times.
```
3. Refresh the page.

The number should increment. 
```
Hello World! I have been seen 2 times. 
```
4. Switch to another terminal window, and type docker image ls to list local images.  

Listing images at this point should return redis and web.  

```
$ docker image ls
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
composetest_web         latest              e2c21aa48cc1        4 minutes ago       93.8MB
python                  3.4-alpine          84e6077c7ab6        7 days ago          82.5MB
redis                   alpine              9d8fa9aa0e5b        3 weeks ago         27.5MB
```

You can inspect images with docker inspect <tag or id>.  

5. Stop the application, either by running docker-compose down from within your project directory in the second terminal, or by hitting CTRL+C in the original terminal where you started the app.  


### Step 5: Edit the Compose file to add a bind mount  
Edit docker-compose.yml in your project directory to add a bind mount for the web service:  

```
version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
    volumes:
     - .:/code
  redis:
    image: "redis:alpine"
```

The new volumes key mounts the project directory (current directory) on the host to /code inside the container, allowing you to modify the code on the fly, without having to rebuild the image.  

### Step 6: Re-build and run the app with Compose  
From your project directory, type docker-compose up to build the app with the updated Compose file, and run it.  

```
$ docker-compose up
Creating network "composetest_default" with the default driver
Creating composetest_web_1 ...
Creating composetest_redis_1 ...
Creating composetest_web_1
Creating composetest_redis_1 ... done
Attaching to composetest_web_1, composetest_redis_1
web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
...
```

Check the Hello World message in a web browser again, and refresh to see the count increment. 

### Step 7: Update the application  

Because the application code is now mounted into the container using a volume, you can make changes to its code and see the changes instantly, without having to rebuild the image.  

Change the greeting in app.py and save it. For example, change the Hello World! message to Hello from Docker!:  

```
return 'Hello from Docker! I have been seen {} times.\n'.format(count)
```

Refresh the app in your browser. The greeting should be updated, and the counter should still be incrementing.  

### Step 8: Experiment with some other commands  
If you want to run your services in the background, you can pass the -d flag (for “detached” mode) to docker-compose up and use docker-compose ps to see what is currently running:  

```
$ docker-compose up -d
Starting composetest_redis_1...
Starting composetest_web_1...

```
```
$ docker-compose ps
```

Name                 Command            State       Ports  
-------------------------------------------------------------------  
composetest_redis_1   /usr/local/bin/run         Up  
composetest_web_1     /bin/sh -c python app.py   Up      5000->5000/tcp  
The docker-compose run command allows you to run one-off commands for your services. For example, to see what environment variables are available to the web service:  

```
$ docker-compose run web env
```
See docker-compose --help to see other available commands. You can also install command completion for the bash and zsh shell, which will also show you available commands.  

If you started Compose with docker-compose up -d, you’ll probably want to stop your services once you’ve finished with them: 

```
$ docker-compose stop
```

You can bring everything down, removing the containers entirely, with the down command. Pass --volumes to also remove the data volume used by the Redis container:  

```
$ docker-compose down --volumes
```

At this point, you have seen the basics of how Compose works.  
