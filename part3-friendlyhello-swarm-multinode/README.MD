Multinode docker swarm with virtual machines nodes using local registry
The commands are run on a Mac with Oracle VirtualBox installed. When installing Docker for Mac, docker-machine is also included

Sources:
https://docs.docker.com/registry/deploying/  
https://docs.docker.com/get-started/part4/   

```bash
# create virtual machines myvm1 and myvm2 with docker. This can take some time
# you can also start VirtualBox application and see the new
# boot2docker virtual machines from there.
docker-machine create --driver virtualbox myvm1
docker-machine create --driver virtualbox myvm2

# list ip address of the new VMs
docker-machine ls
# myvm1 will be the manager and myvm2 will be the worker. Manager runs
# commands on all workers, and workers register on themselves to the manager

# Initilize swarm and make myvm1 the manager.
docker-machine ssh myvm1 "docker swarm init --advertise-addr <myvm1 ip>"
# The above will print a command you can run with a token on a node to add it to the swarm
# run the command on myvm2 with the token to add it to the swarm
docker-machine ssh myvm2 "docker swarm join --token <token> <myvm1 ip>:2377"

# overview of nodes in swarm
docker-machine ssh myvm1 "docker node ls"

# We will now install registry on the manager from where the images will get pulled

# install docker registry on myvm1 by running docker command using ssh
docker-machine ssh myvm1 "docker run -d -p 5000:5000 --restart=always --name registry registry:2"

# Docker requires HTTPS to connect to registry. We must disable it on myvm1 and myvm2 and also the Mac running these commands
# log into myvm1 using ssh
docker-machine ssh myvm1

#disable HTTPS requirement by adding myvm1 IP to myvm1 and myvm2 and restarting docker
sudo cat '{"insecure-registries" : ["<myvm1 ip>:5000"]}' > /etc/docker/daemon.json
sudo /etc/init.d/docker restart
exit
docker-machine ssh myvm2
sudo cat '{ "insecure-registries" : ["<myvm1 ip>:5000"]}' >  /etc/docker/daemon.json
sudo /etc/init.d/docker restart
exit

# Disable on Mac by going to Docker preferences > Daemon > Advanced and adding
# "insecure-registries" : ["<myvm1 ip>:5000"]

# Now we will push an the image from part 1 to the registry
# create the image locally by following part 1, but call it
# <myvm1 ip>:5000/friendlyhello (-t option of build)

# push the image to myvm1 registry
docker push <myvm1 ip>:5000/friendlyhello
# Alternatively remove locally cached images
docker image remove <myvm1 ip>/friendlyhello

# Now the image is added to the registry in myvm1 and we are ready to deploy to the swarm!!

# Instead of running commands using docker-machine ssh, we can configure our local Docker to run all local docker commands on the myvm1 manager. This also allows us to use local compose files directly on manager!
# The following command lists all the environment variables that need to be set in order to configure local docker to run commands on myvm1
docker-machine env myvm1

# Apply the environment variables
eval $(docker-machine env myvm1)

# verify that myvm1 is the active machine by noticing the * sign
docker-machine ls

# Now we can me the swarm manager deploy to all its nodes, but first we must edit the compose file to point to myvm1 registry
# edit docker-compose-swarm.yml by adding myvm1 ip to the image name
vi docker-compose-swarm.yml

# Deploy and call the app getstartedlab (note that this command runs on myvm1)
docker stack deploy -c docker-compose-swarm.yml getstartedlab

# list the containers running on the swarm. Note that replicas are automatically distributed on myvm1 and myvm2!
docker stack ps getstartedlab

#Test your deployed app
curl -4 <myvm1 ip>
curl -4 <myvm1 ip>
curl -4 <myvm1 ip>
curl -4 <myvm1 ip>
curl -4 <myvm1 ip>

#also calls to myvm2 will work because nodes in swarm participate in an ingress routing mesh
curl -4 <myvm2 ip>
curl -4 <myvm2 ip>
curl -4 <myvm2 ip>
curl -4 <myvm2 ip>

# Note that docker takes care of load-balancing your app accross nodes!

# Scale / descale your app by changing the compose file by adding/removing replicas
vi docker-compose-swarm.yml
# redeploy
docker stack deploy -c docker-compose-swarm.yml getstartedlab

# Clean up
# tear duown the stack
docker stack rm getstartedlab
# unset docker environment variables
eval $(docker-machine env -u)
# stop docker machines
docker-machine stop myvm1
docker-machine stop myvm2


```

Alternative: Installing docker-machine on Ubuntu
```bash
# install docker-machine
base=https://github.com/docker/machine/releases/download/v0.14.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
  sudo install /tmp/docker-machine /usr/local/bin/docker-machine
# restart terminal and test docker-machine
docker-machine version
# optionally, fix docker-machine autocompletion in zsh, see sources
sudo apt-get install virtualbox
```
