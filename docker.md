# 官方文档

## get start

```
docker run hello-world
```

```
docker --version
```

### containers

```
docker build -t friendlyhello .
```

```
docker run -p 4000:80 friendlyhello
```

```
docker tag friendlyhello crazycs/friendlyhello:v2
```

```shell
docker build -t friendlyname .  # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyname  # Run "friendlyname" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyname         # Same thing, but in detached mode
docker container ls                                # List all running containers
docker container ls -a             # List all containers, even those not running
docker container stop <hash>           # Gracefully stop the specified container
docker container kill <hash>         # Force shutdown of the specified container
docker container rm <hash>        # Remove specified container from this machine
docker container rm $(docker container ls -a -q)         # Remove all containers
docker image ls -a                             # List all images on this machine
docker image rm <image id>            # Remove specified image from this machine
docker image rm $(docker image ls -a -q)   # Remove all images from this machine
docker login             # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag  # Tag <image> for upload to registry
docker push username/repository:tag            # Upload tagged image to registry
docker run username/repository:tag                   # Run image from a registry
```

### Services

`docker-compose.yml`

```Yaml
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: crazycs/friendlyhello:v2
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"
    networks:
      - webnet
networks:
  webnet:
```

```
docker swarm init
```

```
docker stack deploy -c docker-compose.yml getstartedlab
```

```
docker service ps getstartedlab_web
```

```
docker container ls -q
```

**scale the app**

You can scale the app by changing the `replicas` value in `docker-compose.yml`, saving the change, and re-running the `docker stack deploy` command:

```shell
docker stack deploy -c docker-compose.yml getstartedlab
```

**take down the app and the swarm**

Take the app down with `docker stack rm`:

```shell
docker stack rm getstartedlab
```

Take down the swarm.

```shell
docker swarm leave --force
```

```shell
docker stack ls                                            # List stacks or apps
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file
docker service ls                 # List running services associated with an app
docker service ps <service>                  # List tasks associated with an app
docker inspect <task or container>                   # Inspect task or container
docker container ls -q                                      # List container IDs
docker stack rm <appname>                             # Tear down an application
docker swarm leave --force      # Take down a single node swarm from the manager
```

**swarm**

```shell
docker-machine create --driver virtualbox myvm1 # Create a VM (Mac, Win7, Linux)
docker-machine create -d hyperv --hyperv-virtual-switch "myswitch" myvm1 # Win10
docker-machine env myvm1                # View basic information about your node
docker-machine ssh myvm1 "docker node ls"         # List the nodes in your swarm
docker-machine ssh myvm1 "docker node inspect <node ID>"        # Inspect a node
docker-machine ssh myvm1 "docker swarm join-token -q worker"   # View join token
docker-machine ssh myvm1   # Open an SSH session with the VM; type "exit" to end
docker node ls                # View nodes in swarm (while logged on to manager)
docker-machine ssh myvm2 "docker swarm leave"  # Make the worker leave the swarm
docker-machine ssh myvm1 "docker swarm leave -f" # Make master leave, kill swarm
docker-machine ls # list VMs, asterisk shows which VM this shell is talking to
docker-machine start myvm1            # Start a VM that is currently not running
docker-machine env myvm1      # show environment variables and command for myvm1
eval $(docker-machine env myvm1)         # Mac command to connect shell to myvm1
& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression   # Windows command to connect shell to myvm1
docker stack deploy -c <file> <app>  # Deploy an app; command shell must be set to talk to manager (myvm1), uses local Compose file
docker-machine scp docker-compose.yml myvm1:~ # Copy file to node's home dir (only required if you use ssh to connect to manager and deploy the app)
docker-machine ssh myvm1 "docker stack deploy -c <file> <app>"   # Deploy an app using ssh (you must have first copied the Compose file to myvm1)
eval $(docker-machine env -u)     # Disconnect shell from VMs, use native docker
docker-machine stop $(docker-machine ls -q)               # Stop all running VMs
docker-machine rm $(docker-machine ls -q) # Delete all VMs and their disk images
```



# 常用命令

启动

```shell
docker run -it --name xxx ubuntu /bin/bash  #启动一个交互式容器
docker start xxx #启动已经停止运行的容器
docker run --name test_log -d ubuntu /bin/sh -c "while true;do echo hello crazycs ;sleep 1;done"  #启动一个守护式容器
```

**attach**

```shell
docker attach xxxx
#如果退出容器的shell  ， 容器会再次停止运行
```

**查看容器日志**

```Shell
docker logs [ -f [ t ] ]  xxx   # t  是在日志输出前加上时间戳
docker logs --tail 10 xxx  #获取日志的最后10行

```

**docker 日志驱动**

```Shell
docker run --log-driver="syslog" --name test_log -d ubuntu /bin/sh -c "while true;do echo hello crazycs ;sleep 1;done"  #启动一个守护式容器，并设置日志驱动为syslog , 此时无法通过 docker logs xx 看到该容器的日志输出
```

**查看容器内的进程**

```shell
docker top xxx
```

**docker 统计信息**

```shell
docker stats xxx xxxx xxxx #查看多个容器的统计信息，包括CPU,memory , network ...
```

**在容器内运行新进程**

```shell
docker exec xxx touch /etc/new_file		#新建一个文件
docker exec -it xxx /bin/bash 		#新建一个bash交互进程
```

**停止守护式进程容器**

```shell
docker stop xxx				
```

**自动重启容器**

由于某种错误导致容器运行停止，可以通过 —restart 标志位设置自动重启，默认docker 不自动重启容器

```shell
docker run --restart=always --name test_log -d ubuntu /bin/sh -c "while true;do echo hello crazycs ;sleep 1;done"  #容器停止后自动重启容器
docker run --restart=on-failure:5 --name test_log -d ubuntu /bin/sh -c "while true;do echo hello crazycs ;sleep 1;done"  #容器停止后自动重启容器,只限5次

```

**查看容器的具体信息** ， 其配置信息，名称，命令，网络配置等

```shell
docker inspect xxx
docker inspect --format='{{.State.Running}}' xxx
docker inspect --format='{{.NetworkSettings.IPAddress }}' xxx
```

**删除容器**

```shell
docker rm xxx  #xxx 可以说容器ID
```

**删除镜像**

```
docker rmi xxx
```

查看运行中的容器**

```
docker ps
```

**查看docker 端口映射情况**

```shell
docker port xxx
```



# DockerFile 



