Using swarm with local registry

Sources: 
https://docs.docker.com/registry/deploying/  
https://docs.docker.com/get-started/part3/

```bash
#download and start registry 
docker run -d -p 5000:5000 --restart=always --name registry registry:2
#retag before pushing local image "friendlyhello" to registry 
#must use url of registry in tag 
docker image tag friendlyhello localhost:5000/my-friendlyhello
#push to registry
docker push localhost:5000/my-friendlyhello
#remove locally cached images to test fetching from registry
docker image remove friendlyhello
docker image remove localhost:5000/my-friendlyhello
#test pull from registry
docker pull localhost:5000/my-friendlyhello
#remove
docker image remove localhost:5000/my-friendlyhello

#### using swarm to deploy service with docker-compose.yml
#### make sure that docker-compose.yml has correct image name
#### localhost:5000/my-friendlyhello:latest
#initialize docker swarm
docker swarm init
#deploy app and give it name "getstartedlab"
#this will run 5 containers instances of our image (per yml file)
docker stack deploy -c docker-compose.yml getstartedlab
#check the running app (service)
docker service ls
#list all 5 containers in our app (postfix web is due to yml)
docker service ps getstartedlab_web
#or
docker container ls
#test - (may need to restart machine after using swarm first time...)
#note that hostname changes because of replicas
curl -4 http://localhost
curl -4 http://localhost
curl -4 http://localhost
curl -4 http://localhost
#scale the app by changing replicas i docker-compose.yml
vi docker-compose.yml 
#redeploy changes
docker stack deploy -c docker-compose.yml getstartedlab
#view new replicas
docker container ls
#Finish!
#take the app down
docker stack rm getstartedlab
#take swarm down
docker swarm leave --force
```



