Source: https://docs.docker.com/get-started/part2/

```bash
#build image with tag friendlyhello
docker build -t friendlyhello .
#list images
docker image ls
#run
docker run -p 4000:80 friendlyhello
#test
curl http://localhost:4000
#stop
docker container stop <hash>
```
