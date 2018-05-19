Source: https://docs.docker.com/get-started/part5/#persist-the-data


Here, the docker compose file from previous part is updated with two new services:
- visualizer - this visualises docker containers on swarm nodes
- redis - to persist each visit and read number of visits on our web app

Together, the webapp, redis and visualizer form a stack. A stack consists of multiple services that depend on each other. Our web app needs the redis in order to store.

Check out comments in compose file to view highlights of important parts.

```bash
# create dir data on home folder of myvm1 - this is where the redis will
# save its files. See the compose file for more details
docker-machine ssh myvm1 "mkdir ./data"

# configure shell to run on myvm1
eval $(docker-machine env myvm1)

# deploy the stack
docker stack deploy -c docker-compose.yml getstartedlab
# go to visualizer url to view its gui: <myvm1 ip>:8080
# Here you can see how all the containers are deployed on myvm1 and myvm2

# call the web app to see that counts are registered
curl -4 <myvm1 ip>
curl -4 <myvm1 ip>
curl -4 <myvm1 ip>
curl -4 <myvm1 ip>
curl -4 <myvm2 ip>
curl -4 <myvm2 ip>
```
